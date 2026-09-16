# @pipeworx/who-euro

Health statistics for the **WHO European Region** — 2,657 indicators across 22
datasets and 53 countries, many with series running back to the 1970s. Sourced
from the WHO Regional Office for Europe Data Warehouse API v5.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1576+ live data sources.

## Tools

| Tool | Answers |
|---|---|
| `who_euro_search_measures` | Find an indicator by name — "tuberculosis incidence", "life expectancy" |
| `who_euro_measure_data` | The time series for one indicator, by country and year |
| `who_euro_list_datasets` | The 22 collections (HFA, European mortality, ENHIS, HBSC, AMR…) |
| `who_euro_dataset_measures` | Every indicator inside one dataset |
| `who_euro_countries` | Countries and country groups, with their codes |
| `who_euro_measure_metadata` | Definition, data source, units and footnotes |

## Two things to know before editing

**The host is not in WHO's own docs.** The published API specification writes
the base as `http://HOST/api/v5/…` — a literal placeholder. The real host
(`dw.euro.who.int`) appears only in the gateway site's page markup. Anything
under `gateway.euro.who.int/api/…` returns 404.

**There is no text search.** `?search=`, `?q=`, `?name=` all return
`400 invalid parameter`. The only filter is:

```
?filter=ATTRIBUTE:CODES_LIST;ATTRIBUTE:CODES_LIST
        e.g. COUNTRY:UKR;YEAR:2015-2020;SEX:ALL
```

…over structured dimension codes, never over names. So searching by wording
happens in this pack, over the full measures list, which is fetched once per
isolate and memoised. `CODES_LIST` also supports `*`, `$blank`, ranges
(`1999-2004`), and nearest-year lookups (`~2001`).

## Indicator names work, not just codes

`who_euro_measure_data` accepts an indicator **name** as well as a code. This is
not a convenience — codes like `HFA_1` are unguessable, and a model asked for
data will invent one. A live routing test for *"tuberculosis incidence in
Ukraine"* produced `HFA_468`, which is rural sewage access. Responses carry
`measure_resolution` saying what the name resolved to, plus other candidates
when the wording is ambiguous.

## Coverage note

WHO/Europe advertises "8,000+ indicators". The v5 API returns **2,657** measures
in English — the larger figure appears to count across languages, datasets, or
retired series. The pack reports what the API actually serves.

## Auth

None.

## Known upstream quirk

`dw.euro.who.int` presents an **incomplete certificate chain** — leaf only, no
intermediate (Sectigo DV R36). `curl` completes it from its own store; strict
TLS clients such as Node's undici reject it with
`UNABLE_TO_VERIFY_LEAF_SIGNATURE`. If a connection fails, the pack says so
explicitly rather than returning a bare "fetch failed", which would read like a
WHO outage.

## Data sources

- WHO/Europe Data Warehouse API — <https://dw.euro.who.int/api/v5/version>
- API specification — <https://gateway.euro.who.int/en/api/specification/>
- European Health Information Gateway — <https://gateway.euro.who.int/>

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "who-euro": {
      "url": "https://gateway.pipeworx.io/who-euro/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/who-euro/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1576+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "who-euro": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-who-euro"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-who-euro
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Who Euro data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
