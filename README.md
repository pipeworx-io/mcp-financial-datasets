# mcp-financial-datasets

Financial Datasets AI MCP — wraps financialdatasets.ai

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1858+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

| Tool | Description |
|------|-------------|
| `fd_institutional_holders` | Who holds <ticker> (institutional 13F holders) — the institutional investors (funds, asset managers) that report holding a stock in their latest SEC Form 13F. Returns investor name, shares held, market value, and report period. Example: fd_institutional_holders({ ticker: "AAPL", limit: 25, _apiKey: "your-key" }) |
| `fd_investor_portfolio` | What does fund <investor> hold (institutional 13F portfolio) — every long position an institutional filer reported in its latest SEC Form 13F. Identified by the fund's 10-digit zero-padded SEC CIK (e.g. Berkshire Hathaway = 0001067983). Returns ticker, shares, market value, and report period per position. Example: fd_investor_portfolio({ filer_cik: "0001067983", limit: 50, _apiKey: "your-key" }) |
| `fd_insider_transactions` | Insider (Form 4) transactions for <ticker> — recent buys/sells by officers, directors, and 10%+ owners as reported on SEC Form 3/4/5. Returns insider name, title, transaction type, shares, price, date, and board-director flag. Example: fd_insider_transactions({ ticker: "AAPL", limit: 25, _apiKey: "your-key" }) |
| `fd_filings` | Recent SEC filings for <ticker> — the company's latest EDGAR filings (10-K, 10-Q, 8-K, 20-F, 6-K). Returns filing type, report period, filing date, and document URL. Optionally filter by filing_type. Example: fd_filings({ ticker: "AAPL", filing_type: "8-K", limit: 20, _apiKey: "your-key" }) |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "financial-datasets": {
      "url": "https://gateway.pipeworx.io/financial-datasets/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/financial-datasets/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1858+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/fd_institutional_holders`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-financial-datasets` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/financial-datasets/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "financial-datasets": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-financial-datasets"]
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
ask_pipeworx({ question: "your question about Financial Datasets data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
