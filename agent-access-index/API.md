# Agent access index — tool reference

3 tools on `https://agentwares-access-index.vercel.app/mcp`, generated from a live `tools/list`
call against the deployed server. Errors are JSON objects carrying `code`, `cause`, `fix` and
`retryable`, so a client can decide whether to retry without parsing prose.

See [README.md](README.md) for authentication and a call you can paste into a terminal.

## `access_check_url`

Says whether an AI agent may fetch a URL, on what terms, and at what price. Reads robots.txt for a generic agent and for every AI user-agent the operators document, the Content-signal and Content-Usage directives, any RSL licence the origin links, and the live response to an honestly-identified request. Verdicts: `allowed` (it served us), `allowed_if_identified` (the written policy permits this but bot management refused an anonymous request — signing as a named crawler is the documented route), `charged` (there is a price, quoted where the rail will show one), `blocked` (the site disallows the agents that would want this, with nothing to buy), and `unknown` (the origin did not answer, or the URL is not there). Never evades: no CAPTCHA solving, no proxies, no spoofed user-agents. Refusals come back as findings with the evidence attached.

| Parameter         | Type    | Required | Description                                                                                             |
| ----------------- | ------- | -------- | ------------------------------------------------------------------------------------------------------- |
| `url`             | string  | yes      | Absolute http(s) URL of the page to check, e.g. https://example.com/article                             |
| `includeEvidence` | boolean | no       | When true, the answer includes every request made, with status and headers (default false: it is large) |
| `skipWellKnown`   | boolean | no       | When true, the five well-known probes are skipped; faster, and those files are almost always absent     |

Annotations: `readOnlyHint`, `idempotentHint`, `openWorldHint`

## `access_explain_verdicts`

Returns the five verdicts access_check_url can give, what each one means for an agent that has just been refused, and the next move for each, plus what each confidence level rests on. Free, makes no network request, and returns the same answer every time: it is the key to reading access_check_url results.

_Takes no arguments._

Annotations: `readOnlyHint`, `idempotentHint`

## `access_list_agents`

Returns the AI crawler tokens this index reads robots.txt rules for — only tokens their operators document — plus the widely-written tokens that no operator sends, which make a robots rule unenforceable, and the list of verdict names. Useful when writing or auditing a robots.txt.

_Takes no arguments._

Annotations: `readOnlyHint`, `idempotentHint`
