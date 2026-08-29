# Building a 3‑Agent Pi/Herdr System: Manager → Coder → Reviewer

This guide sets up a self-looping multi-agent coding workflow using **[pi](https://github.com/badlogic/pi-mono)** (the terminal coding agent) and **[herdr](https://herdr.dev)** (the terminal workspace manager for AI agents), bridged by the **pi-herdr** extension. One agent (the **Manager**) sits in an orchestrator role and shuttles work between a **Coder** and a **Reviewer**, looping automatically until the Reviewer approves. Shared memory (specs, decisions, review history) lives as plain markdown notes in an **Obsidian vault** that all three agents read and write directly.

> Everything here is plain markdown files + CLI tools. Obsidian itself is optional — it's a viewer/graph UI on top of the vault folder. The agents never need Obsidian open; they just read/write files in that folder.

---

## 1. How the three roles fit together

```
                 ┌───────────────────────────┐
                 │         MANAGER            │
                 │  (pi, orchestrator mode,   │
                 │   pi-herdr extension)      │
                 └─────────────┬──────────────┘
                                │ delegates task
                    ┌───────────┴────────────┐
                    ▼                         ▼
            ┌───────────────┐        ┌────────────────┐
            │     CODER      │──────▶│    REVIEWER     │
            │ (herdr pane)   │  diff  │  (herdr pane)   │
            └───────────────┘        └────────────────┘
                    ▲                         │
                    │      NEEDS_WORK          │
                    └─────────────────────────┘
                                │ SHIP
                                ▼
                     Manager marks task done,
                     writes result to vault
```

- **Manager** — an orchestrator-only pi session. It cannot edit files, run bash, or touch code itself; it can only spawn/prompt/read other agents (via `pi-herdr` tools) and read/write the Obsidian vault. Its whole job is: read the task → send to Coder → send the diff to Reviewer → read the verdict → loop or finish.
- **Coder** — a normal pi (or claude/codex) agent spawned in its own herdr pane, with full file/bash access to the repo. It implements or revises the change.
- **Reviewer** — a second agent, ideally a **different model/CLI** than the Coder (e.g. Coder = `pi`, Reviewer = `claude`), spawned in its own pane. It only reads the diff and returns a verdict: `SHIP`, `NEEDS_WORK`, or `MAJOR_RETHINK`, plus reasons.
- **Obsidian vault** — a folder of `.md` files that all three read/write for durable memory: the task spec, running decisions log, and every review round. This is what lets the Manager "remember" prior rounds without re-explaining the whole history in every prompt.

---

## 2. Prerequisites

Install these once, in order:

```bash
# 1. pi — the host coding agent
npm install -g @earendil-works/pi-coding-agent
pi --version
pi            # run once, then /login to configure a model + API key
```

```bash
# 2. herdr — the terminal workspace manager
# Install from https://herdr.dev for your platform, then:
herdr --version
herdr status         # "server: not running" until you launch it
herdr                # launches the workspace + starts the local server
```

> **macOS note:** launch `herdr` from a normal terminal, not via `brew services`/launchd — a launchd-managed server inherits a minimal PATH with no `node`, so spawned `pi` panes die silently. Just run `herdr` from your shell.

```bash
# 3. pi-herdr — the extension that lets pi orchestrate herdr panes
pi install npm:@andrewjacop/pi-herdr
```

This registers the extension globally, so it loads in every pi session — including ones you spawn later. Restart pi (or run `/reload`) if a session was already open.

You now have three tools available inside any pi session:

| Tool | Purpose |
|---|---|
| `herdr_start_agent` | Launch an agent (`pi`/`claude`/`codex`/`omp`/`custom`) in a new pane |
| `herdr_send_prompt` | Send a prompt to a pane |
| `herdr_read_agent` | Read recent output from a pane |
| `herdr_wait_agent` | Block until a pane reaches idle/working/blocked/done |
| `herdr_list_agents` / `herdr_get_agent` | Inspect the fleet |
| `herdr_stop_agent` | Close a pane (destructive) |
| `herdr_delegate` | Composite: spawn → send → wait → harvest response, one call |

---

## 3. Set up the Obsidian vault as shared memory

Create (or reuse) a vault folder and give it a fixed structure the agents will always write to:

```bash
mkdir -p ~/vaults/pi-herdr-memory/{Tasks,Reviews,Decisions,Roles,Sessions}
```

```
~/vaults/pi-herdr-memory/
├── Tasks/            # one note per task/feature: spec + current status
├── Reviews/           # one note per review round, linked back to the task
├── Decisions/         # durable architectural/style decisions, reused across tasks
├── Roles/             # the three role prompts below (source of truth)
├── Sessions/          # short session summaries the Manager appends after each loop
└── Index.md           # top-level map of the vault
```

Open this folder as a vault in Obsidian if you want the graph/backlink view — it's optional, the agents don't need it open. What matters is that every agent is told (in its role prompt) to read and write notes here using plain file operations, and to link related notes with `[[wiki-links]]` so the structure stays navigable.

### `Index.md`
```markdown
# Pi/Herdr Memory Index

- [[Roles/Manager]] · [[Roles/Coder]] · [[Roles/Reviewer]]
- Active tasks: see Tasks/
- Standing decisions: see Decisions/
```

### `Roles/Manager.md`, `Roles/Coder.md`, `Roles/Reviewer.md`
These are the actual system-prompt text you'll paste in when starting each pi session (Section 5). Keeping them as vault notes means edits to a role are versioned and visible next to the rest of the memory, and any agent can re-read its own role file mid-session if it drifts.

### Task note template — `Tasks/<task-slug>.md`
```markdown
---
status: in-progress   # in-progress | needs-work | shipped
round: 1
---
# <Task title>

## Spec
<what needs to be built/changed>

## Current diff summary
(Coder updates this each round)

## Latest verdict
(Reviewer updates this each round: SHIP / NEEDS_WORK / MAJOR_RETHINK + reasons)

## Links
- [[Reviews/<task-slug>-round-1]]
```

### Review note template — `Reviews/<task-slug>-round-N.md`
```markdown
---
task: "[[Tasks/<task-slug>]]"
round: N
verdict: NEEDS_WORK   # SHIP | NEEDS_WORK | MAJOR_RETHINK
---
## What was reviewed
<diff / summary the coder produced>

## Verdict
<reviewer's reasoning and required changes>
```

---

## 4. Write the three role prompts

Save these into `Roles/*.md` in the vault, and also keep a copy handy to paste when spawning each session.

### `Roles/Manager.md`
```markdown
You are the MANAGER in a 3-agent build loop (Manager / Coder / Reviewer).
You are an orchestrator ONLY: you never edit files or run shell commands yourself.
Your tools are herdr_start_agent, herdr_send_prompt, herdr_read_agent,
herdr_wait_agent, herdr_delegate, and normal file read/write ONLY inside the
Obsidian vault at ~/vaults/pi-herdr-memory.

Loop, for the task described to you:
1. Read or create Tasks/<task-slug>.md in the vault with the spec.
2. Use herdr_start_agent to launch (or reuse) a pane named "coder" (agent: pi).
3. Send the Coder the task spec + any latest reviewer feedback via
   herdr_send_prompt, then herdr_wait_agent until idle.
4. Read the Coder's summary/diff with herdr_read_agent. Append it to the task note.
5. Use herdr_start_agent to launch (or reuse) a pane named "reviewer"
   (agent: claude, so it's a different model than the coder).
6. Send the Reviewer the diff and ask it to reply with exactly one verdict word
   on its own line — SHIP, NEEDS_WORK, or MAJOR_RETHINK — followed by reasons.
7. Wait for the Reviewer, read its verdict, write it to
   Reviews/<task-slug>-round-N.md, and update Tasks/<task-slug>.md
   (status, round, latest verdict).
8. If verdict is SHIP: mark the task "shipped" in the vault, tell the user
   it's done, and stop.
   If NEEDS_WORK: increment round, send the reviewer's feedback back to the
   Coder, and repeat from step 3.
   If MAJOR_RETHINK: stop the loop, mark status "needs-work", and surface the
   reviewer's concerns to the user instead of retrying automatically.
9. Cap at 5 rounds — if still not SHIP after 5, stop and report to the user
   rather than looping forever.

Never approve work yourself. Only the Reviewer's verdict can close the loop.
```

### `Roles/Coder.md`
```markdown
You are the CODER in a 3-agent build loop. You have full file/bash access to
the repository. You will receive a task spec and, on later rounds, reviewer
feedback from the Manager.

- Implement the smallest correct change that satisfies the spec/feedback.
- Run the test suite (or relevant tests) before reporting back.
- Reply with a concise summary of what changed and why, plus the diff
  (`git diff`), so the Manager can hand it to the Reviewer.
- Do not mark your own work as done or approved — that's the Reviewer's job.
```

### `Roles/Reviewer.md`
```markdown
You are the REVIEWER in a 3-agent build loop. You did not write this code.
You will receive a diff/summary from the Manager. You have read-only intent:
inspect the diff and the repo as needed, but do not modify files.

Check for: correctness against the stated spec, obvious bugs, missing tests,
style/consistency with the rest of the repo, and security issues.

Reply in this exact format:
<VERDICT>
SHIP | NEEDS_WORK | MAJOR_RETHINK
</VERDICT>
<REASONS>
- bullet list of concrete, actionable reasons
</REASONS>

Use SHIP only if you would merge this as-is. Use MAJOR_RETHINK only if the
approach itself is wrong, not just the implementation details.
```

---

## 5. Start the loop

**Terminal 1 — herdr server:**
```bash
herdr
```

**Terminal 2 — the Manager (orchestrator pi):**
```bash
cd my-project
pi
```
Then, inside that pi session, turn it into an orchestrator (per pi-subagents' orchestrator mode, or simply by pasting the Manager role) and kick off the task:

```
Load your role from ~/vaults/pi-herdr-memory/Roles/Manager.md and follow it.

Task: "<describe the feature/bugfix here>"

Create the task note in the vault, then start the Manager→Coder→Reviewer loop
using herdr_start_agent / herdr_send_prompt / herdr_wait_agent, spawning the
coder as a "pi" agent and the reviewer as a "claude" agent, until the Reviewer
says SHIP or the round cap is hit.
```

The Manager will now:
1. Write `Tasks/<task-slug>.md`.
2. `herdr_start_agent` → spawns a `pi` pane named `coder`, pastes in `Roles/Coder.md` + the spec.
3. `herdr_wait_agent` on `coder` until idle, `herdr_read_agent` to harvest its diff summary.
4. `herdr_start_agent` → spawns a `claude` pane named `reviewer`, pastes in `Roles/Reviewer.md` + the diff.
5. `herdr_wait_agent` on `reviewer`, reads the verdict, writes it to `Reviews/…round-N.md`.
6. If `NEEDS_WORK`, sends the feedback back to `coder` and repeats; if `SHIP`, stops and reports.

You can watch it happen live — herdr shows three labeled panes (`[coder]`, `[reviewer]`, plus your Manager terminal), and you can attach to any pane at any time to look over an agent's shoulder or type into it directly, without breaking the Manager's automation.

---

## 6. Why heterogeneous Coder/Reviewer matters

Using different underlying CLIs/models for Coder vs. Reviewer (e.g. `pi` writes, `claude` reviews) avoids a same-model agent rubber-stamping its own reasoning. `pi-herdr` supports this natively — `herdr_start_agent` takes an `agent` field of `pi`, `claude`, `codex`, or `omp` (opencode), and each spawned pane is an independent CLI process, so you can freely mix vendors. If you'd rather keep everything same-vendor for cost/latency reasons, that's fine too — just change `agent: "claude"` to `agent: "pi"` in the Reviewer step.

---

## 7. Practical tips

- **Completion detection can lag.** `pi-herdr` self-reports pi's own idle/working state reliably; for non-pi agents (claude/codex) it falls back to herdr's auto-detection plus polling. If a wait ever hangs, you can manually unstick a pane: `herdr pane report-agent <pane> --source manual --agent pi --state idle`.
- **Interrupting a stuck pane:** `herdr pane send-keys <pane> C-c`.
- **Keep the vault as the single source of truth**, not the chat history — the Manager's context window will fill up over many rounds, but `Tasks/*.md` and `Reviews/*.md` persist regardless, so a fresh Manager session can resume a task by just reading the vault.
- **Round cap.** Always give the Manager a hard cap (Section 5 uses 5) so a disagreeing Coder/Reviewer pair can't loop forever silently — you want it to surface to you instead.
- **Parallel tasks.** Because each task gets its own note and each Coder/Reviewer pair its own named panes, you can run several Manager loops for different tasks concurrently — just give each pane set a unique name prefix (e.g. `coder-authfix`, `reviewer-authfix`) so they don't collide.
- **Costs.** Every round spawns two full agent turns (Coder + Reviewer) on top of the Manager's own orchestration calls — for larger changes this adds up quickly, so a tight, well-scoped task spec pays off more than usual.

---

## 8. Minimal checklist

- [ ] `pi`, `herdr`, and `pi-herdr` installed; `herdr status` shows the server running
- [ ] Obsidian vault folder created with `Tasks/ Reviews/ Decisions/ Roles/ Sessions/`
- [ ] `Roles/Manager.md`, `Roles/Coder.md`, `Roles/Reviewer.md` written
- [ ] Manager pi session started, pointed at its role file and the task
- [ ] Coder and Reviewer panes spawn automatically via `herdr_start_agent`
- [ ] Loop runs until `SHIP`, `MAJOR_RETHINK`, or the round cap — never silently forever
