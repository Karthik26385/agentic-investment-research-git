# Tool Directory

All MCP servers and built-in tools used by this project, with connection details, available operations, and pipeline integration notes.

---

## Directory Structure

```
tool/
├── README.md                    ← this file (master index)
├── mcp/
│   ├── gmail/
│   │   ├── config.json          ← connection endpoint, auth type, UUID
│   │   └── README.md            ← available tools + pipeline usage
│   ├── google-drive/
│   │   ├── config.json
│   │   └── README.md
│   ├── vercel/
│   │   ├── config.json
│   │   └── README.md
│   └── robinhood/
│       ├── config.json          ← setup instructions (not yet connected)
│       └── README.md
└── builtin/
    ├── websearch/
    │   └── README.md
    └── webfetch/
        └── README.md
```

---

## Tool Status Summary

| Tool | Type | Status | Pipeline Role |
|------|------|--------|--------------|
| [Gmail](mcp/gmail/README.md) | MCP Server | ✅ Connected | Daily email reports (Step 7) |
| [Google Drive](mcp/google-drive/README.md) | MCP Server | ⚠️ Needs Auth | Not yet used in pipeline |
| [Vercel](mcp/vercel/README.md) | MCP Plugin | ⚠️ Needs Auth | Not used in pipeline |
| [Robinhood](mcp/robinhood/README.md) | MCP Server | 🟡 Registered (restart to activate) | Live quotes, stop monitoring, order execution |
| [WebSearch](builtin/websearch/README.md) | Built-in | ✅ Active | All 5 signal agents; Step F price checks |
| [WebFetch](builtin/webfetch/README.md) | Built-in | ✅ Active | Structured data retrieval from known URLs |

---

## Registered MCP Servers

```bash
# Robinhood — added 2026-06-23 (restart session to activate)
claude mcp add robinhood-trading --transport http https://agent.robinhood.com/mcp/trading

# Verify all registered servers
claude mcp list
```

After adding a new server, update the relevant `tool/mcp/<name>/config.json` with the live endpoint and status.

---

## Pipeline Integration Map

```
orchestrator.md
  │
  ├── WebSearch      → Steps 1, 3, F (price validity)
  ├── WebFetch       → Steps 1, F (structured data)
  ├── Robinhood MCP  → Step F (live quotes), Step 8 (close prices), EXECUTION orders
  └── Gmail MCP      → Step 7 (email report)
```
