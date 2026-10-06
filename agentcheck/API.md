# agentcheck — tool reference

8 tools on `https://agentwares-agentcheck.vercel.app/api/mcp`, generated from a live `tools/list`
call against the deployed server. Errors are JSON objects carrying `code`, `cause`, `fix` and
`retryable`, so a client can decide whether to retry without parsing prose.

See [README.md](README.md) for authentication and a call you can paste into a terminal.

## `agentcheck_get_status`

Current status of a public monitored target: overall state, uptime over 24h/7d/30d, last check time, last nightly scores, open incidents and the badge/status URLs. Use the owner (GitHub login) and target slug from the status page URL https://agentwares-agentcheck.vercel.app/<owner>/<slug>. No API key needed.

| Parameter | Type   | Required | Description                                   |
| --------- | ------ | -------- | --------------------------------------------- |
| `owner`   | string | yes      | GitHub login of the target's owner, e.g. demo |
| `slug`    | string | yes      | target slug, e.g. demo                        |

Annotations: `readOnlyHint`, `idempotentHint`

## `agentcheck_get_pricing`

Machine-readable pricing for agentcheck: tiers with monthly USD price, target limits, check interval and features, plus per-run add-ons. Same data as /pricing.json. No API key needed.

_Takes no arguments._

Annotations: `readOnlyHint`, `idempotentHint`

## `agentcheck_create_target`

Enroll something to monitor: an http endpoint (JSON or OpenAI-style chat), a remote MCP server (Streamable HTTP url) or an A2A agent (origin with /.well-known/agent-card.json). Pass `checks` to create checks in the same call (POST /api/v1/probe proposes three). Returns the target id, the public status page, the badge SVG URL and a README snippet. The first check runs on the next scheduled tick, within five minutes; call agentcheck_run_now to run immediately. Free tier: 1 target, hourly; Starter+: 5-minute checks. Requires an API key.

| Parameter      | Type                                   | Required | Description                                                                                                                                       |
| -------------- | -------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`         | string                                 | no       | display name; defaults to the host                                                                                                                |
| `slug`         | string                                 | no       | URL slug for /<owner>/<slug>; defaults to a slug of the name                                                                                      |
| `kind`         | `"http"` \| `"mcp"` \| `"a2a"`         | yes      | http (JSON or chat endpoint), mcp (Streamable HTTP url, or npx/uvx package spec), a2a (agent card)                                                |
| `url`          | string                                 | no       | endpoint URL (http/mcp) or the agent's origin (a2a)                                                                                               |
| `packageSpec`  | string                                 | no       | mcp only: `npx @org/server` / `uvx server` — runs on the mcpcheck runner, not the scheduled checks                                                |
| `agentCardUrl` | string                                 | no       | a2a: explicit agent card URL when it is not at /.well-known/agent-card.json                                                                       |
| `format`       | `"openai_chat"` \| `"json"` \| `"get"` | no       | http: openai_chat (POST /chat/completions body), json (raw POST), get (plain fetch)                                                               |
| `headers`      | object                                 | no       | extra request headers                                                                                                                             |
| `auth`         | object \| object                       | no       | credential the prober sends to your endpoint: {type:'bearer',token} or {type:'header',name,value}. Stored encrypted and never returned or logged. |
| `modelVar`     | string                                 | no       | the model your agent runs on (e.g. claude-sonnet-5); enables model-drift re-runs                                                                  |
| `modelHeader`  | string                                 | no       | request header your endpoint accepts to override the model (drift re-runs try the new model)                                                      |
| `corpusUrl`    | string                                 | no       | RAG targets: public corpus URL for the nightly groundedness scorer (Pro)                                                                          |
| `alerts`       | object                                 | no       | where to send an incident when a check starts failing. Any combination; omit it and incidents are visible only on the status page.                |
| `isPublic`     | boolean                                | no       | public status page + badge (default true)                                                                                                         |
| `checks`       | array of object                        | no       | checks to create right away (the probe proposes three)                                                                                            |

Annotations: `openWorldHint`

## `agentcheck_add_check`

Add a check to one of your targets. A check runs an input (http path/prompt, mcp tool call, a2a message) on a schedule and judges the answer with a golden: exact, contains, regex and json_schema cost nothing; rubric and baseline use the LLM judge (baseline = same outcome as the last known-good answer). Returns the check id. Requires an API key.

| Parameter     | Type                                                                              | Required | Description                                                                                                                                                                                    |
| ------------- | --------------------------------------------------------------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`        | string                                                                            | yes      | short name shown on the status page                                                                                                                                                            |
| `kind`        | `"http"` \| `"mcp_tools_list"` \| `"mcp_tool_call"` \| `"a2a_task"` \| `"custom"` | yes      | what to run: http (GET path or POST prompt), mcp_tools_list, mcp_tool_call (tool + args), a2a_task (message)                                                                                   |
| `input`       | object                                                                            | no       | http: { path?, method?, body?, prompt? } · mcp_tool_call: { tool, args } · a2a_task: { message }                                                                                               |
| `golden`      | object                                                                            | yes      | how the answer is judged. exact / contains / regex / json_schema are free and deterministic; rubric and baseline call the LLM judge, and baseline compares against the last known-good answer. |
| `intervalSec` | integer                                                                           | no       | seconds between runs; the tier's interval is the floor (Free hourly, Starter+ 5 min)                                                                                                           |
| `targetId`    | string                                                                            | yes      | id from agentcheck_create_target or GET /api/v1/targets                                                                                                                                        |

## `agentcheck_run_now`

Run every check of one of your targets immediately (outside the schedule) and return pass/fail per check with latency, judge cost and any incident opened or closed. Use it right after enrolling, or to confirm a fix. Requires an API key.

| Parameter  | Type   | Required | Description |
| ---------- | ------ | -------- | ----------- |
| `targetId` | string | yes      | target id   |

Annotations: `openWorldHint`

## `agentcheck_record`

Record one production interaction with your agent (the prompt or messages, the final answer, the tools it called, optionally the model) as a trace on a target. Call it from your agent or from a proxy in front of it after each task; promote a good trace with agentcheck_promote_trace to replay it nightly and catch regressions. Requires an API key.

| Parameter   | Type                                | Required | Description                                                                     |
| ----------- | ----------------------------------- | -------- | ------------------------------------------------------------------------------- |
| `targetId`  | string                              | yes      | id from agentcheck_create_target or GET /api/v1/targets                         |
| `name`      | string                              | no       | short label, e.g. 'refund for order A-1029'                                     |
| `input`     | string \| array of object \| object | yes      | the prompt, the messages array, or an object with prompt/messages               |
| `output`    | string                              | yes      | the agent's final answer                                                        |
| `toolCalls` | array of object                     | no       | tool calls in order                                                             |
| `model`     | string                              | no       | the model that produced this answer, so drift re-runs can compare across models |

## `agentcheck_promote_trace`

Turn an imported or recorded trace into a replayable check whose golden is the recorded outcome (tool sequence + final-answer rubric). The check runs daily and in the nightly replay (Pro). Returns the check. Requires an API key.

| Parameter | Type   | Required | Description                                                         |
| --------- | ------ | -------- | ------------------------------------------------------------------- |
| `traceId` | string | yes      | trace id from agentcheck_record or POST /api/v1/targets/{id}/traces |

Annotations: `idempotentHint`

## `agentcheck_list_incidents`

Open and recent incidents across your targets (or one target): when they opened/closed, the failing check and the cause. Requires an API key.

| Parameter  | Type   | Required | Description                                                                 |
| ---------- | ------ | -------- | --------------------------------------------------------------------------- |
| `targetId` | string | no       | narrow to one target; omit for incidents across every target on the account |

Annotations: `readOnlyHint`, `idempotentHint`
