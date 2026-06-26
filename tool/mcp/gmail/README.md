# Gmail MCP

**Status:** Connect via claude.ai Settings → Integrations → Gmail
**Endpoint:** `https://gmailmcp.googleapis.com/mcp/v1`
**Connector UUID:** `YOUR_GMAIL_CONNECTOR_UUID` — set after connecting via claude.ai
**Auth:** OAuth2 via Google, managed by claude.ai

---

## Setup

1. Go to [claude.ai](https://claude.ai) → Settings → Integrations → Gmail → Connect
2. Complete the Google OAuth2 flow
3. Note your connector UUID from the integration page
4. Update `tool/mcp/gmail/config.json` → replace `YOUR_GMAIL_CONNECTOR_UUID`
5. Update `CLAUDE.md` → replace `YOUR_GMAIL_CONNECTOR_UUID` in the Data Sources section

---

## Pipeline Role

Used in **Step 7** of the orchestrator to create the daily email draft after each simulation run.

Triggered when `mode.email_report = true` (enabled in SIMULATION and EXECUTION; disabled in RESEARCH and BACKTESTING).

---

## Available Tools

| Tool | Description | Used in Pipeline |
|------|-------------|-----------------|
| `create_draft` | Create a new Gmail draft | ✅ Step 7 — daily brief draft |
| `list_drafts` | List existing drafts | — |
| `get_thread` | Fetch a full email thread | — |
| `search_threads` | Search emails by query | — |
| `create_label` | Create a Gmail label | — |
| `delete_label` | Delete a Gmail label | — |
| `update_label` | Update label name/color | — |
| `list_labels` | List all labels | — |
| `label_message` | Apply label to a message | — |
| `unlabel_message` | Remove label from a message | — |
| `label_thread` | Apply label to a thread | — |
| `unlabel_thread` | Remove label from a thread | — |

### Notable Limitation

**`send_message` is not available.** Drafts are created automatically by the pipeline, but the user must open Gmail and send the draft manually.

---

## Step 7 Email Format

```
Subject: [SIMULATION] Daily Brief — YYYY-MM-DD | {REGIME} | {SUB_LABEL}

Sections:
  - Regime classification + signal scores (table)
  - Market snapshot (index moves)
  - Portfolio actions today (EXIT / HOLD / NEW)
  - Critical watchpoints
  - Portfolio snapshot (P&L, drawdown, win rate)
```

---

## Reconnect Instructions

If the Gmail MCP loses its connection:

1. Open [claude.ai](https://claude.ai) → Settings → Integrations
2. Find **Gmail** → click **Reconnect**
3. Complete the Google OAuth2 flow

```bash
claude mcp list   # verify status
```
