# AI Market Research Agent

**Autonomous investment research and paper trading system powered by Claude Code.**

An AI-driven hedge fund simulation engine that runs a full daily research cycle: macro signal detection → regime classification → thesis generation → adversarial validation → sector exposure control → risk-calibrated paper execution → portfolio tracking.

> This is a decision-support and simulation framework. It is not a trading bot and does not execute real trades without explicit human confirmation in EXECUTION mode.

---

## What It Does

- Runs a structured **9-step daily pipeline** driven by markdown workflow files interpreted by Claude
- Classifies the market regime (BULL / SIDEWAYS / BEAR) from 5 parallel signal agents
- Generates trade theses, attacks each one adversarially (counter-thesis gate), and enforces sector concentration caps
- Tracks a paper portfolio ($100 default) with full stop/target management and equity curve
- Supports **4 operating modes**: RESEARCH, SIMULATION, EXECUTION, BACKTESTING
- Tags every pipeline run with a **Session ID (SID)** so multiple same-day runs coexist cleanly
- Sends daily summary email via Gmail MCP (optional)
- Uses **Robinhood MCP** for live equity quotes, earnings calendars, and EXECUTION orders

---

## Architecture

```
User command ("start SIMULATION", "Start BACKTESTING 10 Days", etc.)
         │
         ▼
  orchestrator.md  ←  reads state/mode.json  |  generates SID (HHMMSS)
         │
         ├── [BACKTESTING] → backtesting_engine.md (N-day historical simulation)
         │
         └── [RESEARCH / SIMULATION / EXECUTION]
               ├── Step 0.5  portfolio_review.md      → check open positions (live prices)
               ├── Step 1    5× signal_agents/*.md     → macro / earnings / tech / sentiment / geo
               ├── Step 2    regime engine             → BULL / SIDEWAYS / BEAR
               ├── Step 3    daily_market_brief.md + top5_selection.md
               ├── Step 4    counter_thesis.md         → WEAK/MEDIUM/STRONG gate
               ├── Step 4.5  sector_exposure_control.md → 40% soft / 50% hard cap
               ├── Step 5    trade_execution_thinking.md → paper or real trade
               ├── Step 6    write [DATE]_[SID]_*.md output files
               ├── Step 7    Gmail MCP email report
               └── Step 8    post_market_review.md (4 PM ET)
```
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/b4ce15e6-522d-461a-bed1-f388beaa6e1f" />

---

## Quick Start

```bash
# 1. Clone the repo
git clone https://github.com/YOUR_USERNAME/agentic-investment-research-git.git
cd agentic-investment-research-git

# 2. Set up MCP servers (Robinhood + Gmail) — see MCP Setup below

# 3. Open in Claude Code and run
# In the Claude Code chat window, type:
start SIMULATION
```

---

## MCP Setup

### Robinhood MCP (live quotes + EXECUTION orders)

```bash
claude mcp add robinhood-trading --transport http https://agent.robinhood.com/mcp/trading
```

Then restart Claude Code to activate. Verify with `claude mcp list`.

See [`tool/mcp/robinhood/README.md`](tool/mcp/robinhood/README.md) for full setup and safety gate documentation.

> EXECUTION mode (real orders) requires this. RESEARCH and SIMULATION work without it via WebSearch fallback.

### Gmail MCP (daily email reports)

1. Go to [claude.ai](https://claude.ai) → Settings → Integrations → Gmail → Connect
2. Complete the Google OAuth2 flow
3. Update `tool/mcp/gmail/config.json` with your connector UUID
4. Update `CLAUDE.md` — replace `YOUR_GMAIL_CONNECTOR_UUID` with your actual UUID

Email reports are optional. Set `"email_report": false` in `state/mode.json` to disable.

---

## Commands

All operations are triggered by natural language in the Claude Code chat window.

See [`commands/system_prompts.md`](commands/system_prompts.md) for the full reference.

**Most common:**

| Command | When to Use |
|---------|-------------|
| `start SIMULATION` | Every morning — daily paper trading run |
| `start RESEARCH` | Analysis without committing paper capital |
| `Start BACKTESTING 10 Days` | After strategy changes — stress-test the gates |
| `start SIMULATION with full deployable $100` | Reset portfolio to fresh $100 |
| `Run portfolio review` | Mid-session stop check |
| `Run post market review` | After 4 PM ET — lock in close prices |
| `switch to EXECUTION` | Go live (requires ≥10 sim days + ≥45% win rate) |

> **Tip:** Append `; Granted all permission to bash and powershell` to skip tool-use permission prompts.

---

## Modes

| Mode | Trigger | What It Does | Output |
|------|---------|-------------|--------|
| **RESEARCH** | `start RESEARCH` | Full analysis pipeline. No portfolio tracking. | `FinalResearchoutput/research/` |
| **SIMULATION** | `start SIMULATION` | Paper trading with $100 virtual portfolio. | `FinalResearchoutput/simulation/` |
| **EXECUTION** | `switch to EXECUTION` | Live orders via Robinhood MCP. Human confirmation required. | `FinalResearchoutput/live/` |
| **BACKTESTING** | `Start BACKTESTING N Days` | Historical simulation over last N trading days (v1.0: plumbing test). | `FinalResearchoutput/backtesting/` |

Active mode is set in `state/mode.json` → `active_mode` field.

---

## State Files

All persistent state lives in `state/` as JSON. These files survive across sessions.

| File | Purpose |
|------|---------|
| `state/mode.json` | Active mode + per-mode configuration |
| `state/portfolio_ledger.json` | All trades, positions, P&L, equity curve |
| `state/regime_state.json` | Current regime + 30-day history + signal weights |
| `state/signal_cache.json` | Latest 5-agent outputs (stale if date ≠ today → auto re-run) |
| `state/sector_exposure.json` | Capital per sector; enforces 40%/50% caps |
| `state/thesis_queue.json` | Today's thesis candidates |
| `state/performance_log.json` | Signal accuracy history (partially manual) |
| `state/trade_history.json` | Backtest trade records (tagged `BACKTEST`) |
| `state/signal_performance.json` | Backtest signal accuracy (tagged `BACKTEST`) |

> **Never delete state files.** They carry the full audit trail. The repo ships clean template versions.

---

## Requirements

- **Claude Code** (Sonnet 4.6 or later) — the system runs entirely within Claude Code sessions
- **Robinhood account** — for live quotes (RESEARCH/SIMULATION) and orders (EXECUTION)
- **Gmail account** — for daily email reports (optional)
- No Python, Node.js, or other runtimes required

---

## Honest Limitations

- **Not a real-time system.** Stops are checked at session start only — not monitored continuously.
- **Backtesting (v1.0) has look-ahead bias.** Claude's training data contains historical outcomes. Results are plumbing/logic tests, not statistical edge validation.
- **Signal weights never auto-update.** The learning loop is aspirational; every session starts with equal weights (0.20 each).
- **$100 paper capital.** Sufficient for mechanics testing; psychologically and contextually insignificant.
- **Robinhood does not support short selling.** SHORT theses in EXECUTION mode are converted to inverse ETFs or put options (Step 8b).

For a complete honest assessment, see [`readMe/Solution_Document.md`](readMe/Solution_Document.md) Section 12.

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| v1.0 | 2026-06-25 | Initial public release — 4 modes, SID system, backtesting engine v1.0, 9-step pipeline |

---

## License

MIT — see [LICENSE](LICENSE)
"# agentic-investment-research-git" 
