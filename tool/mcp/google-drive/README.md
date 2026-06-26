# Google Drive MCP

**Status:** ⚠️ Needs Authentication  
**Endpoint:** `https://drivemcp.googleapis.com/mcp/v1`  
**Auth:** OAuth2 via Google (same account as Gmail connector)

---

## Current State

The Google Drive MCP is registered but requires re-authentication. It is **not currently used** in any pipeline step.

---

## Planned Pipeline Integration

Once authenticated, Google Drive can serve as a persistent cloud archive for daily output files:

| Pipeline Point | Use |
|----------------|-----|
| After Step 6 (output files written) | Upload `FinalResearchoutput/simulation/YYYY-MM-DD/` to Drive |
| Cross-device access | View research output from mobile or other machines |
| Backup | Redundant copy of state files (`state/*.json`) |
| Sharing | Share research summaries with collaborators |

---

## Expected Tools (once authenticated)

| Tool | Description |
|------|-------------|
| `list_files` | List files in a folder |
| `get_file` | Fetch file content |
| `search_files` | Search by name or content |
| `create_file` | Upload a new file |
| `update_file` | Update existing file content |
| `delete_file` | Remove a file |
| `list_folders` | List Drive folders |
| `create_folder` | Create a folder |
| `share_file` | Set sharing permissions |

---

## Reconnect Instructions

1. Open [claude.ai](https://claude.ai) → Settings → Integrations
2. Find **Google Drive** → click **Reconnect**
3. Complete the Google OAuth2 flow (same Google account as Gmail)
4. Verify in Claude Code: `claude mcp list`

---

## Proposed Folder Structure in Drive

```
AI-MarketResearch-Agent/
├── FinalResearchoutput/
│   └── simulation/
│       └── YYYY-MM-DD/
│           ├── YYYY-MM-DD_Portfolio_Review.md
│           ├── YYYY-MM-DD_Market_Regime.md
│           ├── YYYY-MM-DD_Daily_Brief.md
│           ├── YYYY-MM-DD_Top5_Scored.md
│           ├── YYYY-MM-DD_Counter_Thesis.md
│           ├── YYYY-MM-DD_Trade_Execution.md
│           ├── YYYY-MM-DD_Master_Summary.md
│           └── YYYY-MM-DD_Portfolio_Log.md
└── state/
    └── (periodic snapshots of state JSON files)
```
