# AI_prompts_DB

A curated collection of LLM prompts and prompt-engineering guides — each one documented with what it does, when to use it, required inputs, and a ready-to-copy template.

## Contents

| File | What it's for |
|---|---|
| [`SEO_Prompt_Library.md`](./SEO_Prompt_Library.md) | Six SEO/content prompts (full article writer, keyword research, meta descriptions, content strategist, keyword-rich content, article length optimizer), each with purpose, required inputs, step-by-step usage, pitfalls, and a compact copy-paste template. |
| [`n_8_n_workflow_prompt_guide.md`](./n_8_n_workflow_prompt_guide.md) | A prompt for designing production-ready n8n automation workflows, plus a 14-section fill-in-the-blank specification template (architecture, node-by-node config, error handling, testing kit, deployment checklist). |
| [`pi-herdr-multiagent.MD`](./pi-herdr-multiagent.MD) | A setup guide (not a single prompt) for a 3-agent Manager → Coder → Reviewer coding loop using `pi` and `herdr`, including the three role prompts and an Obsidian-vault memory structure. |

## How to use these

1. Open the relevant file and find the prompt/template you need.
2. Fill in the bracketed placeholders (`[INSERT ...]`) with your specifics.
3. Paste the completed prompt into your LLM of choice.
4. Where a file gives both a full template and a "compact" version, start with the compact one for quick use and switch to the full version when you need more control.

## Contributing

New prompts and guides are welcome. To keep the collection consistent:

- One file per prompt family/topic (e.g. `SEO_Prompt_Library.md`, not one file per individual prompt).
- For each prompt, document: **Purpose**, **When to use**, **Required inputs**, **Expected output**, **Step-by-step usage**, **Best practices**, **Pitfalls**, and a ready-to-copy **Example template**.
- Include a table of contents at the top of longer files.
- Prefer a short "compact" version of each template alongside the full one, for quick reuse.
- Use fenced code blocks for the actual prompt/template text so it can be copy-pasted directly.

Open a PR with your new or updated file — no need to touch the others.

## License

See [`LICENSE`](./LICENSE).