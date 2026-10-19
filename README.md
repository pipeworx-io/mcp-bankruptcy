# Bankruptcy — US business bankruptcy filings

Business bankruptcy filings from US federal bankruptcy courts: **1.55 million
filings**, searchable by company name, court and date. No API key.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1847+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Auth

None.

## What this is, and what it is not

This is **docket metadata**: that a case exists, who filed, in which court, when
it was filed, when it terminated, and which judge was assigned. It contains
**no filings** — no petitions, schedules, motions or orders. Those live in
RECAP, a separate corpus that is not keyless.

An agent that has this data knows a company filed. It does not know what the
company said.

## Consumer bankruptcy is deliberately excluded

Of CourtListener's 71,677,647 dockets, **35,312,051 are individual bankruptcy
filings** — half the corpus. They carry no business signal and are PII-adjacent,
so they are excluded.

The docket export has **no chapter column**, so business and consumer are
separated by a legal-entity-suffix test on the debtor name (`Inc`, `LLC`,
`Corp`, `Holdings`, `Construction`, and so on), applied only inside bankruptcy
courts. Two consequences worth knowing before relying on a negative result:

- **A sole proprietorship filing under a personal name will not appear here.**
- "No filings found" means "no *business* filing matched", never "this entity
  has never filed".

## Tools

| Tool | Returns |
|---|---|
| `bankruptcy_search` | Filings matching a company name, with court, dates and judge |
| `bankruptcy_recent` | Recent filings, newest first, optionally one court |

Court ids follow CourtListener's convention: `nysb` (S.D.N.Y. Bankruptcy),
`deb` (Delaware), `txsb` (S.D. Tex.), `cacb` (C.D. Cal.), `ilnb` (N.D. Ill.).

## Caveats

- **Quarterly snapshot.** Every response carries `snapshot_date`. Anything filed
  after it is not here.
- **`date_terminated` is 78.3% filled**, so a missing termination date means
  *not recorded*, never *still open*. Responses say this on every row.
- **`nature_of_suit` is sparse** (14.8% corpus-wide) and cannot be relied on to
  filter by case type.
- Case names are as the court recorded them, including abbreviations and
  inconsistent entity suffixes. Search matches substrings for that reason.

## Attribution

Data is CourtListener / Free Law Project, published under the Public Domain
Mark. Free Law Project funds and maintains the corpus. If this pack is useful
to you, <https://free.law/donate/> is where that goes.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "bankruptcy": {
      "url": "https://gateway.pipeworx.io/bankruptcy/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/bankruptcy/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1847+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/bankruptcy_search \
  -H 'Content-Type: application/json' \
  -d '{"company":"Kmart","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/bankruptcy_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "bankruptcy": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-bankruptcy"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-bankruptcy
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Bankruptcy data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
