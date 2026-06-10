# Installing the Auralogs MCP server (agent instructions)

Auralogs is a hosted service. There is nothing to build or run locally; you only configure the client with the remote endpoint and a key.

## Step 1: Get a read key from the user

Ask the user for their Auralogs **read key** (format: `aura_read_...`).

If they don't have one:

1. Sign up at https://auralogs.ai/ (free tier, no credit card).
2. Create a project. The startup modal shows an **MCP/API read key**; copy it.
3. Or create one later: dashboard → **Settings** → **API & MCP keys** → **Read keys** → **Create**. The plaintext key is shown once.

## Step 2: Configure the client

- Endpoint: `https://mcp.auralogs.ai/mcp`
- Transport: Streamable HTTP
- Header: `Authorization: Bearer aura_read_YOUR_KEY`

Cline (`cline_mcp_settings.json`):

```json
{
  "mcpServers": {
    "auralogs": {
      "type": "streamableHttp",
      "url": "https://mcp.auralogs.ai/mcp",
      "headers": {
        "Authorization": "Bearer aura_read_YOUR_KEY"
      }
    }
  }
}
```

Claude Code:

```bash
claude mcp add --transport http auralogs https://mcp.auralogs.ai/mcp \
  --header "Authorization: Bearer aura_read_YOUR_KEY"
```

Cursor (`.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "auralogs": {
      "url": "https://mcp.auralogs.ai/mcp",
      "headers": {
        "Authorization": "Bearer aura_read_YOUR_KEY"
      }
    }
  }
}
```

## Step 3: Verify

```bash
curl -X POST https://mcp.auralogs.ai/mcp \
  -H "Authorization: Bearer aura_read_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"test","version":"0"}}}'
```

Expect HTTP 200 with `result.serverInfo.name = "auralogs"`. A 401 means the key is wrong or revoked.

The server is read-only: 7 tools (`list_projects`, `get_project`, `list_logs`, `search_logs`, `get_log`, `list_analyses`, `get_analysis`), no write operations. Rate limit is 60 requests/minute per key.
