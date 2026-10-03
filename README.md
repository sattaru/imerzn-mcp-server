# Imerzn MCP Server

![MCP Spec](https://img.shields.io/badge/MCP%20spec-2025--03--26-blue)
![Transport](https://img.shields.io/badge/transport-Streamable%20HTTP-green)
![Tools](https://img.shields.io/badge/tools-4-orange)
![Manifest](https://img.shields.io/badge/manifest-live-brightgreen)

**Company-maintained MCP server for [Imerzn](https://imerzn.com) — the travel experiences marketplace built on the Stories, Not Stops® philosophy.**

Imerzn is a travel marketplace connecting explorers with authentic local experiences across 100+ countries. Every listing is backed by verified Localites — named, in-country guides who curate and lead experiences — or by trusted Bokun partner operators. This MCP server exposes Imerzn's live catalog directly to AI assistants: Claude, ChatGPT, and any MCP-compatible client can search experiences, surface local guides, fetch full activity details, and browse editorial blog content — all in real time, with no API key required.

## Quick Start

### Claude Desktop

Add to your `claude_desktop_config.json` (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS, `%APPDATA%\Claude\claude_desktop_config.json` on Windows):

```json
{
  "mcpServers": {
    "imerzn": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://imerzn.com/mcp"]
    }
  }
}
```

Restart Claude Desktop. Ask: *"What experiences are available in Kyoto, Japan?"*

### Claude Code

```bash
claude mcp add imerzn -- npx -y mcp-remote https://imerzn.com/mcp
```

Or add directly to your project's `.claude/settings.json`:

```json
{
  "mcpServers": {
    "imerzn": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://imerzn.com/mcp"]
    }
  }
}
```

### mcp-remote direct

```bash
npx -y mcp-remote https://imerzn.com/mcp
```

## Tools

| Tool | Description | Schema |
|------|-------------|--------|
| [`search_experiences`](#search_experiences) | Search the Imerzn catalog by country, state, or city. Returns summaries with price, image, and detail URL. | [tools/search_experiences.json](tools/search_experiences.json) |
| [`list_local_guides`](#list_local_guides) | List verified Localite guides and curator partners for a location. | [tools/list_local_guides.json](tools/list_local_guides.json) |
| [`get_experience_detail`](#get_experience_detail) | Retrieve full detail for a single experience by ID (native numeric, Bokun `p-N-aM`, or Oddball slug). | [tools/get_experience_detail.json](tools/get_experience_detail.json) |
| [`list_blog_articles`](#list_blog_articles) | Browse Imerzn editorial blog articles by free-text title search. | [tools/list_blog_articles.json](tools/list_blog_articles.json) |

### search_experiences

Search Imerzn's catalog across three provider types (native, Bokun partner operators, Oddball games). `country` is required; `state` and `city` narrow further.

```json
{
  "country": "Japan",
  "state": "Kyoto",
  "number_of_rows": 10
}
```

### list_local_guides

List verified Imerzn Localites for a location. All parameters are optional — omit all to list globally.

```json
{
  "country": "Portugal",
  "city": "Lisbon"
}
```

### get_experience_detail

Retrieve full experience detail. Three ID formats are supported:
- **Native**: `"42"` (numeric string)
- **Bokun**: `"p-8085-a866982"` (partner-activity format)
- **Oddball**: `"tokyo-murder-mystery"` (slug string)

```json
{
  "id": "p-8085-a866982"
}
```

### list_blog_articles

Browse editorial blog content. `search_text` matches against the blog title. Omit to list the latest articles.

```json
{
  "search_text": "Morocco",
  "number_of_rows": 5
}
```

## Live Surface

The server is live at `https://imerzn.com/mcp`. All calls use JSON-RPC 2.0 over Streamable HTTP (POST).

**List available tools:**

```bash
curl -s -X POST https://imerzn.com/mcp \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"tools/list","id":1}' | jq '.result.tools[].name'
```

**Search experiences in France:**

```bash
curl -s -X POST https://imerzn.com/mcp \
  -H "Content-Type: application/json" \
  --data '{
    "jsonrpc": "2.0",
    "method": "tools/call",
    "id": 2,
    "params": {
      "name": "search_experiences",
      "arguments": {"country": "France", "city": "Paris", "number_of_rows": 3}
    }
  }' | jq '.result'
```

**Get a blog article list:**

```bash
curl -s -X POST https://imerzn.com/mcp \
  -H "Content-Type: application/json" \
  --data '{
    "jsonrpc": "2.0",
    "method": "tools/call",
    "id": 3,
    "params": {
      "name": "list_blog_articles",
      "arguments": {"search_text": "Japan", "number_of_rows": 5}
    }
  }' | jq '.result'
```

## Protocol

| Property | Value |
|----------|-------|
| Transport | Streamable HTTP |
| Protocol | JSON-RPC 2.0 |
| MCP spec version | 2025-03-26 |
| Endpoint | `https://imerzn.com/mcp` |
| Authentication | None — no API key required |
| Rate limit | 60 requests / minute / IP |
| Content-Type | `application/json` |

## Links

| Resource | URL |
|----------|-----|
| Imerzn marketplace | https://imerzn.com |
| Live manifest | https://imerzn.com/.well-known/mcp/manifest.json |
| OpenAPI 3.1 spec | https://imerzn.com/openapi.json |
| MCP endpoint | https://imerzn.com/mcp |
| GitHub org | https://github.com/Siesta-Global-Inc |

## License

MIT — see [LICENSE](LICENSE)

Copyright 2026 Siesta Global Inc. (Imerzn)
