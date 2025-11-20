# n8n Workflow Prompt — Guide & Template

> **What this file is** — concise explanation of the prompt and what the resulting markdown contains.

This document explains the prompt (an instruction for an n8n workflow architect) and supplies a ready-to-use Markdown template that you can paste into your editor. It includes the original prompt (as a script block) and a complete, structured template to create an n8n workflow spec that meets the deliverables and constraints.

---

## Original prompt (script)

```
Act like an expert n8n workflow architect who builds reliable, low-code AI-powered automations.
Objective
Design a complete n8n workflow for the specific use case below, optimized for clarity, reliability, and low cost, ready to paste into n8n with minimal edits.
Use case
[Describe precisely what must be automated, who triggers it, required inputs, success criteria, and any SLAs.]
Deliverables (follow this order)

Workflow overview: 3-5 sentences on what the workflow does and the business outcome.
Architecture (text diagram): Trigger → key nodes → outputs, including error paths and retries.
Node-by-node spec: For each node, provide Name, Type, Purpose, Exact configuration (fields/params), Expressions, Connections (incoming/outgoing).
AI agent (only if needed): Model choice, system prompt, tools/functions, context window strategy, token/cost guardrails.
Data handling: Input schema + validation, transformations, output schema, storage (tables/collections/keys).
Integrations: For each external API/service-auth method, endpoints, request/response shapes, rate limits, and failure handling.
Logic & control flow: IF/Switch conditions, loops, backoff + retry rules, circuit breakers, and fallbacks.
Testing kit: 3-5 test cases with sample inputs, expected outputs, and edge cases.
Optimization: Latency/cost tips, caching/memoization, batching, parallelization, and scalability notes.
Deployment checklist: credentials, env vars, webhook URLs, permissions, monitoring/logging, alerts, rollout steps.

Constraints

Prefer native n8n nodes over Code nodes; only add code when strictly necessary and include concise snippets.
Use clear, consistent node names and variable keys.
Include observability: execution logs, metrics, and alerting points.
Keep secrets in credentials/env; never hardcode.

Implementation specifics to include

Exact JSON bodies for HTTP nodes, headers, and auth settings.
All expressions for data mapping (e.g., {{$json.field}}).
Error handling patterns (Try/Catch, conditional reroute, dead-letter queue).
Minimal reproducible examples for tricky parts.

Format
Use concise markdown with headings and bullet points. No filler text. End with a final “How to build” step-by-step list (1-10) that a junior can follow.
Take a deep breath and work on this problem step-by-step.
```

---

## What the prompt asks (brief)

- **Role:** act as an expert n8n workflow architect.
- **Goal:** produce a production-ready, low-code n8n workflow spec for a specific use case (user must provide the use case text inside the bracketed section).
- **Format & constraints:** strict order of deliverables, prefer native nodes, include exact HTTP node JSON, expressions, observability, and a build checklist. End with a 1–10 step build list.


---

## How to use this template

1. Replace the `Use case` placeholder with a specific description (who triggers, inputs, outputs, SLA).
2. For each deliverable section below, fill the placeholders with details from your use case.
3. Where examples are provided, adapt the exact JSON and expressions to your real endpoints and fields.


---

# Template — n8n Workflow Specification

> **Keep this order**. Be concise. Use native nodes when possible.

## 1) Workflow overview

- **What it does (3–5 sentences):**
  - One-liner: *What trigger starts the workflow and the primary business outcome.*
  - Short explanation of how n8n transforms the inputs to outputs.
  - Success criteria (e.g., 99% processed within 2 minutes, delivery within SLA).

## 2) Architecture (text diagram)

`Trigger --> Validate Input --> Enrichment (API) --> Decision (Switch) --> Main Action(s) --> Persist --> Notify --> Success`

- **Error path:** `Any node error -> Try/Catch node -> Dead-letter queue (DB or queue) + Alert (Slack/email)`.
- **Retries:** exponential backoff (0s, 10s, 30s) x3 before dead-letter.

## 3) Node-by-node spec (repeat per node)

**Template for a node**
- **Name:** `HTTP - Enrich User`
- **Type:** HTTP Request (n8n native)
- **Purpose:** Call external enrichment API to add missing metadata.
- **Exact configuration:**
  - **Method:** `POST`
  - **URL:** `https://api.example.com/v1/enrich`
  - **Authentication:** `Header Auth` — `Authorization: Bearer {{$credentials.my_api.token}}`
  - **Headers:** `Content-Type: application/json`
  - **Body (JSON):**
    ```json
    {
      "email": "{{$json["email"]}}",
      "name": "{{$json["name"]}}"
    }
    ```
  - **Response format:** `JSON`
- **Expressions used:**
  - `{{$json["email"]}}`, `{{$json["id"]}}`
- **Connections:**
  - **Incoming:** `Validate Input`
  - **Outgoing (success):** `Switch - Has Data`
  - **Outgoing (error):** `Try/Catch - Enrich Fail`

> Repeat for every node in the workflow, including Trigger, Set, Function (only if necessary), HTTP Request, Switch, Wait, Delay, Webhook, MySQL/Postgres, MongoDB, and Email/Slack.

## 4) AI agent (only if needed)

- **Model choice:** e.g., `gpt-4o-mini` for short responses or `gpt-4o` if large context needed (choose cheapest adequate model).
- **System prompt:** concise, role-limited instruction and safety guardrails.
- **Tools/functions exposed:** `tool_http_get`, `tool_db_query` (if n8n function nodes call external tool endpoints).
- **Context strategy:** send only essential context (truncate or summarize older messages), keep token window under X tokens.
- **Cost guardrails:** max tokens per call, temperature 0.0–0.2 for deterministic replies, rate limits per minute.

## 5) Data handling

- **Input schema (JSON Schema):**
  ```json
  {
    "type": "object",
    "required": ["id","email"],
    "properties": {
      "id": {"type": "string"},
      "email": {"type":"string","format":"email"},
      "metadata": {"type":"object"}
    }
  }
  ```
- **Validation:** `IF missing required -> respond 400 and stop`.
- **Transformations:** mapping rules with expressions, e.g. `{{$json["created_at"] | date("yyyy-MM-dd")}}` (show exact n8n expression where needed).
- **Output schema:** final object stored/sent.
- **Storage:** collection/table names, primary keys, TTL for retry entries, dead-letter table `dlq_workflowname`.

## 6) Integrations

For each external service list:
- **Name & auth method:** e.g., `Stripe — Bearer token (OAuth or API key stored in credentials)`.
- **Endpoints & bodies:** provide exact HTTP node JSON (see template above).
- **Request/response shapes:** show expected sample payloads and key fields to read (e.g., `id`, `status`, `amount`).
- **Rate limits & handling:** e.g., `100 requests/min` -> implement local rate limiter or queue; on 429 -> respect `Retry-After` header and requeue.

## 7) Logic & control flow

- **Switch conditions (example):**
  - `{{$json["status"]}} === "needs_review"` -> route to human-in-the-loop.
- **Loops:** use `SplitInBatches` for batch processing with `batchSize` set; ensure `Wait` node if hitting rate limits.
- **Backoff & retries:** exponential backoff, max attempts = 3.
- **Circuit breaker:** if failure rate > 50% over 1-minute window, open breaker for 5 minutes and send alert.
- **Fallbacks:** if enrichment fails -> continue with partial data + log + notify.

## 8) Testing kit

- **Test case 1 — Happy path:**
  - **Input:** valid payload with id+email.
  - **Expected:** enrichment returned, DB row created, Slack success message.
- **Test case 2 — Missing required field:**
  - **Input:** payload without email.
  - **Expected:** validation node returns 400 and stops; no downstream calls.
- **Test case 3 — External API 500:**
  - **Input:** valid payload; enrichment API returns 500.
  - **Expected:** retry applied x3 then move to DLQ and alert created.
- **Test case 4 — Rate limit scenario:**
  - **Input:** burst of 500 requests/min.
  - **Expected:** batching/queueing; `Retry-After` respected; no data loss.

## 9) Optimization

- **Latency/cost tips:** choose smaller model, low temperature, limit context; cache enrichment results by `email` for 24h.
- **Caching/memoization:** Redis or DB cache with TTL.
- **Batching:** use `SplitInBatches` with `batchSize=20` for downstream APIs that accept bulk.
- **Parallelization:** allow upstream to run parallel but limit parallel downstream HTTP concurrency.

## 10) Deployment checklist

- **Credentials & env vars:** list keys (e.g., `MY_API_TOKEN`, `DB_CONN`, `SLACK_WEBHOOK`), where to set (n8n credentials or env).
- **Webhook URLs:** confirm and register with external services.
- **Permissions:** ensure service account for DB has insert/update permissions only.
- **Monitoring/logging:** enable n8n execution logs, push errors to Sentry/Logzio, metrics to Prometheus.
- **Alerts:** Slack channel + email on DLQ or circuit-breaker open.
- **Rollout:** deploy to staging first, run test kit, then promote.

## 11) Constraints & Implementation specifics

- **Prefer native nodes:** show code only where unavoidable (include minimal Function/Code snippet when needed).
- **Exact JSON bodies:** provide for each HTTP node in Node-by-node section.
- **Expressions:** always use `{{$json[...]}}` or `{{$node["Name"].json[...]}}` explicitly.
- **Error handling patterns:** include Try/Catch around external calls; on error send to `dlq_workflowname` (DB collection) with `error`, `payload`, `retries`.

## 12) Minimal reproducible example (tricky part)

- **Example:** How to implement exponential retry for an HTTP Request node using native n8n features + a small Function node snippet to compute next backoff:

```javascript
// Function node: compute backoff seconds
const attempt = $json["retries"] || 0;
const backoff = Math.min(60, Math.pow(2, attempt) * 5);
return [{ json: { backoff } }];
```

- Use `Wait` node with `{{$json.backoff}}s` value, then re-call HTTP node.

## 13) Observability & logging

- **Execution logs:** enable `Detailed` execution logging for the workflow in staging.
- **Metrics:** count of success, failures, retries, DLQ entries (exposed via HTTP node to metrics collector).
- **Alert points:** on DLQ write, circuit-breaker open, or >5% failure rate in a 5min window.

## 14) Minimal example: HTTP node exact body

```json
{
  "method": "POST",
  "url": "https://api.example.com/v1/enrich",
  "headers": {
    "Authorization": "Bearer {{$credentials.my_api.token}}",
    "Content-Type": "application/json"
  },
  "body": {
    "email": "{{$json["email"]}}",
    "id": "{{$json["id"]}}"
  }
}
```


---

# How to build (1–10) — step-by-step for a junior

1. Fill the **Use case** description at the top with trigger, inputs, success criteria, and SLA.
2. Create a new workflow in n8n and add the **Trigger** node (Webhook / Schedule / Poll).
3. Add a **Set** or **JSON Validate** node with the input JSON schema and immediate error branch for 400 responses.
4. Add `SplitInBatches` if throughput requires batching; otherwise proceed to `HTTP Request` nodes.
5. Add `Try/Catch` around any external HTTP nodes; on error route to DLQ (DB insert) and send alert.
6. Map fields using expressions `{{$json["field"]}}` and store results to the DB node (Postgres/Mongo).
7. Add a `Slack` or `Email` notification node on success and on DLQ writes.
8. Implement retries using a `Function` + `Wait` + re-try path, with exponential backoff snippet from above.
9. Configure credentials in n8n (no hardcodes): API keys, DB connections, Slack webhook.
10. Run the test kit cases in staging, verify observability (logs & metrics), then promote to production.


---

End of document.

