<p align="center">
  <img src="logo.png" alt="Auralogs" width="120" height="120">
</p>

# Auralogs MCP Server

Hosted, read-only MCP server for [Auralogs](https://auralogs.ai), the logging platform built for AI-assisted teams. Connect Claude Code, Claude Desktop, Cursor, Codex, Cline, or any Streamable HTTP MCP client to your production logs, then ask your agent "what's broken in production right now?" instead of copy-pasting stack traces.

[![Add to Cursor](https://img.shields.io/badge/Add_to-Cursor-1a1a1a)](https://cursor.com/install-mcp?name=auralogs&config=eyJ1cmwiOiJodHRwczovL21jcC5hdXJhbG9ncy5haS9tY3AiLCJoZWFkZXJzIjp7IkF1dGhvcml6YXRpb24iOiJCZWFyZXIgYXVyYV9yZWFkX1lPVVJfS0VZIn19)
[![Docs](https://img.shields.io/badge/docs-docs.auralogs.ai-7c5cff)](https://docs.auralogs.ai/read-api/setup/)

- **Endpoint:** `https://mcp.auralogs.ai/mcp` (Streamable HTTP)
- **Auth:** `Authorization: Bearer aura_read_...` (project-scoped read key)
- **Read-only by design:** no write tools exist. Agents can look, not touch.
- **Rate limit:** 60 requests/minute per key

## Tools

| Tool | Description |
| --- | --- |
| `list_projects` | List projects accessible to the current read key |
| `get_project` | Get a project by id |
| `list_logs` | List logs, filtered by level and time range |
| `search_logs` | Full-text search across log messages |
| `get_log` | Get a single log with full context (stack, metadata) |
| `list_analyses` | List AI analyses, filtered by severity and type |
| `get_analysis` | Get one analysis (diagnosis, suggested fix, PR link) |

## Get a key

1. Sign up free at [auralogs.ai](https://auralogs.ai/) (10,000 logs/month, no credit card).
2. Create a project and copy the **MCP/API read key** from the startup modal, or create one later under **Settings → API & MCP keys → Read keys**.

Keys look like `aura_read_...`, are scoped to one project, and can be revoked instantly from the dashboard.

## Connect

### Claude Code

```bash
claude mcp add --transport http auralogs https://mcp.auralogs.ai/mcp \
  --header "Authorization: Bearer aura_read_YOUR_KEY"
```

### Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "auralogs": {
      "transport": {
        "type": "http",
        "url": "https://mcp.auralogs.ai/mcp",
        "headers": {
          "Authorization": "Bearer aura_read_YOUR_KEY"
        }
      }
    }
  }
}
```

### Cursor

Click the **Add to Cursor** badge above, or add to `.cursor/mcp.json`:

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

### Cline

Add via the **Remote Servers** tab, or in `cline_mcp_settings.json`:

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

### Codex

See [auralogs-codex](https://github.com/auralogs-ai/auralogs-codex) for the Codex plugin, or use any standard remote MCP configuration.

### Anything else

The server speaks standard MCP over Streamable HTTP with bearer auth. Smoke-test with curl:

```bash
curl -X POST https://mcp.auralogs.ai/mcp \
  -H "Authorization: Bearer aura_read_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"test","version":"0"}}}'
```

A `200` with `result.serverInfo.name = "auralogs"` confirms auth and transport.

## Try it

Once connected, ask your agent:

- "What's broken in production right now?"
- "Pull the last 20 error logs and look for patterns."
- "Show me the most recent high-severity analysis."

## Security

- Read keys are **read-only** and scoped to a single project. The MCP surface exposes no write operations.
- Revocation is immediate: revoked keys get `401` on the next request.
- Keys are stored hashed (SHA-256); the plaintext is shown once at creation.

Full documentation: [docs.auralogs.ai/read-api](https://docs.auralogs.ai/read-api/overview/)

## About this repo

This repo holds the public metadata for the hosted Auralogs MCP server: the [MCP Registry](https://registry.modelcontextprotocol.io) `server.json`, install instructions, and listing assets. The server itself is a hosted service; there is nothing to install from here.
