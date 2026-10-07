# MCP liveness — tool reference

3 tools on `https://agentwares-mcp-liveness.vercel.app/mcp`, generated from a live `tools/list`
call against the deployed server. Errors are JSON objects carrying `code`, `cause`, `fix` and
`retryable`, so a client can decide whether to retry without parsing prose.

See [README.md](README.md) for authentication and a call you can paste into a terminal.

## `mcp_liveness_check`

Makes one `initialize` call to an MCP server's endpoint with no credential and reports what a stock client gets. The server is named either by its exact official-registry `name`, or by its endpoint `url` when it is not in the registry or is listed under another name; exactly one of the two is accepted. Outcomes: `usable` (it connected and negotiated a protocol version), `needs_credential` (401 or 403 — by name, the answer also says whether a credential can be obtained in-band or whether a person has to create one), `charged` (402, access is for sale), `unreachable` (not an MCP server there: wrong URL, dead host, or a 200 that is not JSON-RPC), and, for a name only, `local_only` (no remote endpoint; it must be spawned as a process) and `not_listed` (no server with that exact name; its endpoint can still be checked by `url`). A `url` must be public https: private, loopback, link-local, multicast and cloud-metadata addresses are refused, redirects are re-checked, and URL checks are limited to 30 a minute per caller. Useful before relying on a server: roughly two in five remote registry listings do not answer a stock client.

| Parameter             | Type    | Required | Description                                                                                                                                                                                                 |
| --------------------- | ------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`                | string  | no       | The server's exact name in the official MCP registry, namespaced — e.g. `io.github.owner/server`. Not a title. Pass this or `url`, not both.                                                                |
| `url`                 | string  | no       | The server's Streamable HTTP endpoint, e.g. `https://example.com/mcp`, for a server that is not in the registry or is named differently there. Public https only. Pass this or `name`, not both.            |
| `skipCredentialCheck` | boolean | no       | Name checks only: skip the OAuth discovery chain on a 401. Faster (one request instead of up to five) but the answer then cannot say whether a credential is obtainable. A URL check never walks the chain. |

Annotations: `readOnlyHint`, `openWorldHint`

## `mcp_liveness_explain_outcomes`

Returns the six outcomes this server can report — usable, needs_credential, charged, unreachable, local_only and not_listed — what each means for an agent choosing a server from the registry, and the usual next move for each. A static reference: it makes no network request and its answer never changes between calls.

_Takes no arguments._

Annotations: `readOnlyHint`, `idempotentHint`

## `mcp_liveness_drift`

Given a server's exact name in the official MCP registry, reports which of its tools were added, removed, or had their description, input schema or output schema change between a named month and the latest monthly snapshot. Useful when a server that worked before behaves differently now, when a prompt or an integration written against its tools has started failing, or before relying on a cached tool description. A liveness check says a server answers today; this says whether it is still the same server that code was written against. The answer names the changed tools and the kind of change. It does not diff the schemas themselves: the snapshots keep content hashes, not bodies, so `detail` hands back the release-asset location of both full months for a caller that needs the exact diff. A month with no snapshot for that server gets an error naming the months that do have one, never an empty diff that would read as `nothing changed`.

| Parameter | Type   | Required | Description                                                                                                                                                                                          |
| --------- | ------ | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`    | string | yes      | Exact registry name, namespaced — e.g. `io.github.owner/server`. Not a URL and not a title.                                                                                                          |
| `since`   | string | no       | The month to diff from, as `YYYY-MM`. Omit it to diff from the earliest month this server appears in, which is the stable choice — a hard-coded month goes wrong as soon as the next snapshot lands. |

Annotations: `readOnlyHint`, `idempotentHint`
