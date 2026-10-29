# @pipeworx/leadmagic

Find a work email by name + company domain, via LeadMagic's B2B data API
(leadmagic.io). Every email returned is already SMTP-validated on
LeadMagic's side. Part of the `enrich-waterfall` finder rotation (fleet
#2860).

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `leadmagic_find_email(first_name, last_name, domain, _apiKey)` — work
  email, employment verification, and company fields; a confirmed no-match
  is free on LeadMagic's own billing.

## Auth

BYO only. Pass your LeadMagic API key as `_apiKey` (header `X-API-Key`). Get
one at https://leadmagic.io. No platform key (D4).

## Data sources

- `https://api.leadmagic.io/v1/people/email-finder` — email finder.

## Verification notes (fleet #2860, 2026-10-08, no vendor key held)

- **Auth header + live endpoint confirmed by edge probe.** No-key POST to
  `/v1/people/email-finder` returned HTTP 401 with LeadMagic's own structured
  problem+json body naming `X-API-Key` (case-insensitive) or
  `Authorization: Bearer <key>` as the two accepted forms. This pack uses
  `X-API-Key`.
- **Response shape is from vendor docs only** (https://leadmagic.io/docs/api-reference/email-finder,
  read 2026-10-08):
  - hit: `{ email, status: "valid", credits_consumed: 1, message,
    employment_verified, company_name, company_industry, mx_provider }`
  - no match: `{ email: null, status: null, credits_consumed: 0, message,
    first_name, last_name, domain }`
  The no-match case is a fully-populated 200 with `email: null` — this
  pack decides hit-vs-miss on `email`, never the envelope, matching the
  waterfall's house rule (same trap findymail documents for its own API).
- **Not verified (no key held):** the exact set of `status` values beyond
  `"valid"` on a live call (docs only show the one example), and whether a
  malformed domain produces a 4xx or a clean no-match.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "leadmagic": {
      "url": "https://gateway.pipeworx.io/leadmagic/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/leadmagic/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/leadmagic_find_email`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-leadmagic` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/leadmagic/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "leadmagic": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-leadmagic"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Leadmagic data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
