# mcp-sentinel-hub

Sentinel Hub MCP — satellite / earth-observation statistics as JSON (sentinel-hub.com).

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1669+ live data sources.

## Tools

| Tool | Description |
|------|-------------|
| `sentinelhub_ndvi_stats` | NDVI vegetation statistics over an area and time range — returns per-interval mean/min/max/stdev of the vegetation index (NDVI) from Sentinel-2 imagery for a bbox or GeoJSON polygon. Derived numbers only, no imagery. Example: sentinelhub_ndvi_stats({ bbox: [12.44, 41.87, 12.53, 41.93], date_from: "2023-05-01", date_to: "2023-09-01", aggregation: "P1M", _apiKey: "client_id:client_secret" }) |
| `sentinelhub_catalog_search` | Which satellite scenes are available for an area + time — searches the Sentinel-2 catalog (STAC) and returns matching scenes with acquisition datetime and cloud cover. Example: sentinelhub_catalog_search({ bbox: [12.44, 41.87, 12.53, 41.93], date_from: "2023-06-01", date_to: "2023-06-30", limit: 10, _apiKey: "client_id:client_secret" }) |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "sentinel-hub": {
      "url": "https://gateway.pipeworx.io/sentinel-hub/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/sentinel-hub/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1669+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/sentinelhub_ndvi_stats`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "sentinel-hub": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-sentinel-hub"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-sentinel-hub
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Sentinel Hub data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
