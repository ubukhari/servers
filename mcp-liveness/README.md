# MCP liveness

Given a server's name in the official MCP registry, whether a stock client can actually use it: does it answer, does it want a credential, can an agent obtain one, or is the listing pointing at nothing. Free, no key.

|                       |                                                                                  |
| --------------------- | -------------------------------------------------------------------------------- |
| **Service**           | https://agentwares-mcp-liveness.vercel.app                                       |
| **MCP endpoint**      | `https://agentwares-mcp-liveness.vercel.app/mcp` (Streamable HTTP)               |
| **Registry name**     | `io.github.agentwares/mcp-liveness` (v0.3.0)                                     |
| **Tools**             | `mcp_liveness_check`, `mcp_liveness_explain_outcomes`, `mcp_liveness_drift`      |
| **In this directory** | [`server.json`](server.json) · [`llms.txt`](llms.txt) · [tool reference](API.md) |

A registry listing is a claim the publisher typed. Nothing verifies it at publish time, so an agent
picking a server out of a catalogue of 31,272 has no way to know whether the endpoint answers.

One `initialize` call to the endpoint the listing names, no credential and no retry. Six outcomes,
each with a one-line `fix`: `usable`, `needs_credential`, `charged`, `unreachable`,
`local_only`, `not_listed`.

**What a 401 is hiding.** Two servers can both answer 401 and be nothing alike. One publishes the
chain the spec defines — RFC 9728 protected-resource metadata, RFC 8414 authorization-server
metadata, RFC 7591 dynamic client registration — so a client nobody has heard of registers itself
and a person approves once. The other wants a key from a dashboard, or says nothing at all. The
registry lists them identically. On a `needs_credential` answer this walks the chain and reports
which one you are looking at.

**The population.** A random sample of 1,200 of the 18,965 remote listings, 12 September 2026:
63.0% ± 2.7 answer a stock client, 21.8% want a credential, 14.8% are not an MCP server at the
address they gave, 0.4% ask to be paid. Of the 262 wanting a credential, 202 publish the full chain
with a registration endpoint and 45 published nothing a stock client could discover at all.

Every MCP monitor that exists is publisher-side: you enrol your own server and get told when it
breaks. This answers the caller's question, which is the side every agent is on.

What it will not do: never sends a credential, never registers a client, never retries a refusal,
never sends another operator's user-agent. A 200 counts as alive only when it carries a JSON-RPC
result with a `protocolVersion` — parked domains, marketing pages and API gateways all answer 200
to a POST.

No LLM is called, so a check costs nothing to serve and nothing to buy.

## Authentication

None. No key, no signup, no payment, no rate-limit tier to buy.

## First call

Free: the outcome for any server in the registry.

```sh
curl -s "https://agentwares-mcp-liveness.vercel.app/v1/alive?name=io.github.agentwares/mcp-liveness"
```

## MCP client configuration

Claude Desktop, Cursor, Claude Code and anything else that speaks Streamable HTTP:

```json
{
  "mcpServers": {
    "mcp-liveness": {
      "url": "https://agentwares-mcp-liveness.vercel.app/mcp"
    }
  }
}
```

## Endpoints

| Endpoint              | What                                                           |
| --------------------- | -------------------------------------------------------------- |
| `POST /mcp`           | the MCP server (Streamable HTTP), no key                       |
| `GET /v1/alive?name=` | outcome, fix, credential route and the probe behind them       |
| `GET /v1/survey`      | the sampled population numbers, each share with its own margin |
| `GET /llms.txt`       | machine docs                                                   |
| `GET /server.json`    | registry server card                                           |
| `GET /health`         | health                                                         |

## Pricing

Free. This service has no `pricing.json` because it does not charge and has no paid tier to advertise.

## Source and issues

This is a hosted service; its implementation is not in this repository (see the
[root README](../README.md)). **Open an issue here** for anything about the running service — a wrong
answer, a dead endpoint, a tool description that misled your agent, a capability you need.

The files in this directory are generated from the service, so please report a problem rather than
sending a documentation patch.
