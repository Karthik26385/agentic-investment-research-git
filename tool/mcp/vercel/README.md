# Vercel MCP

**Status:** ⚠️ Needs Authentication  
**Endpoint:** `https://mcp.vercel.com`  
**Auth:** OAuth2 via Vercel  
**Plugin ID:** `vercel` (registered in `~/.claude/settings.json` → `enabledPlugins`)

---

## Current State

The Vercel plugin is globally enabled in Claude Code settings but requires re-authentication. It is **not used in this project's pipeline**.

---

## What This Tool Does

The Vercel MCP provides access to Vercel platform capabilities:

| Category | Capabilities |
|----------|-------------|
| Deployments | Create, inspect, promote, rollback deployments |
| Environment Variables | List, add, remove, diff env vars |
| Projects | Link, configure, list Vercel projects |
| Logs | Stream function and build logs |
| Domains | Manage custom domains |
| CI/CD | Trigger builds, inspect pipeline status |

---

## Relevance to This Project

Currently none. This project is a pure markdown/JSON workflow with no web application layer. If a dashboard or web frontend for the Autonomous Hedge Fund were built (e.g., a Next.js portfolio tracker), Vercel MCP would become relevant for deployment management.

---

## Reconnect Instructions

1. Open [claude.ai](https://claude.ai) → Settings → Integrations
2. Find **Vercel** → click **Reconnect**
3. Complete the Vercel OAuth2 flow
4. Verify: `claude mcp list`
