# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Active Mode

**Read `state/mode.json` at the start of EVERY session — no exceptions.**

```
RESEARCH    → Analysis only. No portfolio tracking. Outputs to FinalResearchoutput/research/
SIMULATION  → Paper trading. Portfolio tracked ($100 base). Outputs to FinalResearchoutput/simulation/
EXECUTION   → Live trading. Human confirmation required. Outputs to FinalResearchoutput/live/
BACKTESTING → Historical simulation (last N days). No live trades. Outputs to FinalResearchoutput/backtesting/
```

**Current default: SIMULATION**

---

## Operational Commands

This is a pure workflow/markdown project. There are no build, lint, or test commands. All operations are triggered by natural language:

| User says | Action |
|-----------|--------|
| `start` | Run orchestrator.md → full pipeline in active mode |
| `start SIMULATION` | Switch to SIMULATION, run full pipeline |
| `start RESEARCH` | Switch to RESEARCH, run full pipeline |
| `SIMULATION - Run` | Same as "start SIMULATION" |
| `switch to EXECUTION` | Validate safety gates first, then switch |
| `Start BACKTESTING N Days` | Switch to BACKTESTING, run backtesting_engine.md over last N trading days |
| `Start BACKTESTING` | Switch to BACKTESTING, run with default 10 days |

To switch modes manually: edit `state/mode.json` → change `active_mode` field.

---

## Architecture

This is an Autonomous Hedge Fund Simulation Engine for U.S. stock markets. It has no application code — the entire system is Claude executing markdown workflow files against JSON state, web searches, and MCP tools.

### The 9-Step Pipeline (orchestrator.md enforces this order)

```
Step 0   — Read state/mode.json → determine mode + position caps
             Generate SESSION_ID = current time as HHMMSS (e.g. "142033")
             Print: MODE / Output dir / Portfolio tracking / Max trade size / SESSION_ID
             If mode == BACKTESTING: print Lookback days; route to backtesting_engine.md; skip Steps 0.5–8

Step 0.5 — portfolio_review.md  → review ALL open positions first
             Actions per position: HOLD / ADD / REDUCE / EXIT / TAKE_PROFIT
             Execute EXIT and TAKE_PROFIT immediately before any new analysis
             ADD counts against max_daily_trades
             Also triggers mid-session if any open position moves ±3%

Step 1   — 5 Parallel Signal Agents → write state/signal_cache.json
             macro_agent.md | earnings_agent.md | tech_agent.md
             sentiment_agent.md | geo_agent.md

Step 2   — Regime Engine → state/regime_state.json
             Aggregate score = Σ(agent_weight × agent_score)
             BULL > +2.0 (cap × 1.5) | SIDEWAYS −2.0 to +2.0 (cap × 1.0) | BEAR < −2.0 (cap × 0.5)

Step 3   — Thesis Builder (daily_market_brief + top5_selection) → state/thesis_queue.json
             Regime adjustment: SIDEWAYS/BEAR −1 Sensitivity to Momentum, +1 to Mean Reversion

Step 4   — counter_thesis.md for each PENDING candidate [MANDATORY GATE]
             Step 0b: price reality check (reject if >10% from assumed entry)
             Steps A–E: macro / company / structure / sentiment / timing risks
             Step F: price validity re-check
             Step G: binary gap protocol (R:R must be ≥1:2 at gap price)
             WEAK → APPROVE | MEDIUM → reduce size | STRONG → REJECT

Step 4.5 — sector_exposure_control.md [runs after CT, before execution]
             Map ticker → sector | check state/sector_exposure.json
             Hard cap: 50% per sector | Soft cap: 40% per sector | Max 2 per sector
             Exceeds limit + WEAK + R:R≥1:3 → ALLOW as Concentrated Bet (size −30-50%)
             Exceeds limit + MEDIUM → REDUCE or DELAY
             Exceeds limit + WEAK + R:R<1:3 → REJECT

Step 5   — trade_execution_thinking.md → approved_trades
             RESEARCH: write markdown only
             SIMULATION: update state/portfolio_ledger.json (paper)
             EXECUTION: human confirmation → Robinhood MCP order

Step 6   — Write daily output files to [mode.output_dir]/[YYYY-MM-DD]/
             File names include SESSION_ID: [DATE]_[SID]_[FileName].md
             Multiple runs on the same day coexist in the same folder with distinct SIDs
             Also run master_summary.md → 1–2 best regime-aligned trades only

Step 7   — Email report via Gmail MCP (if mode.email_report = true)
             Subject: [MODE] Daily Brief — [DATE] | [REGIME] | [SENTIMENT]

Step 8   — intraday_monitoring.md (appends to day's files during market hours)
             post_market_review.md (after 4 PM ET) → update all state files
```

### State Files (DO NOT DELETE — persistent across sessions)

| File | Purpose |
|------|---------|
| `state/mode.json` | Active mode + per-mode config (position caps, max trades, etc.) |
| `state/portfolio_ledger.json` | Open positions, closed trades, equity curve, realized/unrealized P&L |
| `state/regime_state.json` | Current regime classification + 30-day signal history + agent weights |
| `state/signal_cache.json` | Latest outputs from all 5 signal agents |
| `state/thesis_queue.json` | Today's thesis candidates with CT ratings and statuses |
| `state/performance_log.json` | Signal accuracy history, CT validation rate, trade outcomes |
| `state/sector_exposure.json` | Current capital allocation per sector; enforces 40%/50% caps |

### Position Sizing Logic

Position size = `position_size_cap_pct` (from mode.json) × regime multiplier × CT rating multiplier:
- Regime BULL → cap × 1.5 | SIDEWAYS → cap × 1.0 | BEAR → cap × 0.5
- WEAK counter-thesis → full base allocation
- MEDIUM counter-thesis → 75% of base
- STRONG counter-thesis → REJECT (no position)

With 2 trades and full $100 capital in SIMULATION (SIDEWAYS): WEAK gets 57.14%, MEDIUM gets 42.86% (ratio derived from 1:0.75 split summing to 100%). In BEAR, multiply all sizes by 0.5 first.

---

## Standing Rules (always apply, no exceptions)

1. **Read state/mode.json first** — every single session
2. **Generate SESSION_ID at Step 0** — HHMMSS format (e.g. "142033"); embed in every output file name; never delete the date folder — runs from different times coexist
3. **portfolio_review.md runs before signal agents** — existing positions reviewed first
4. **counter_thesis.md is mandatory** — no trade card without passing the gate
5. **Step F (Price Validity)** — search actual current price; recalibrate if >10% from assumed entry
6. **Step G (Binary Gap)** — for earnings/FOMC events, recalculate R:R assuming 10% gap; WATCH-ONLY if R:R < 1:2
7. **Max 2 trades/day** in SIMULATION and EXECUTION modes
8. **Final close data preferred** over intraday reads for regime classification
9. **Never modify closed trades** — full audit trail in portfolio_ledger.json
10. **No re-entry same day as stop exit** — let the session end before reconsidering
11. **Trailing stop rules** — raise stop to break-even when unrealized gain ≥7%; raise to entry+5% at ≥12%; sell 25% and trail when price is halfway to target
12. **Regime deterioration on open LONGs** — if regime shifts to BEAR while holding a LONG, reduce to 0.5× original size or EXIT; if regime improves to BULL, ADD up to 1.5× cap
13. **Stale signal cache** — at Step 1, check `signal_cache.json → last_updated`. If not today → invalidate all scores, re-run all 5 agents fresh. Never carry forward prior-session signal scores.
14. **Dollar cap in EXECUTION** — before every real order, verify `entry_price × shares ≤ mode.max_order_usd`. If exceeded, block the order and require explicit human action. Do not silently reduce size.
15. **Sector exposure reconciliation** — at the start of portfolio_review.md, rebuild sector_exposure.json from portfolio_ledger.json open positions. Overwrite stored values if drift > 2%.
16. **No workflow self-modification** — post_market_review.md writes learnings to the daily output file only. Workflow markdown files (orchestrator.md, counter_thesis.md, etc.) may only be edited by a human after explicit review.

---

## Output Naming Convention

All files go to `[mode.output_dir]/[YYYY-MM-DD]/`.

Every file name includes a **SESSION_ID** (`[SID]`) — the wall-clock time the pipeline started, in `HHMMSS` format. Multiple runs on the same date produce distinct files in the same folder; no run ever overwrites another.

```
[DATE]_[SID]_Portfolio_Review.md   ← SIMULATION + EXECUTION only (if open positions)
[DATE]_[SID]_Market_Regime.md
[DATE]_[SID]_Daily_Brief.md
[DATE]_[SID]_Top5_Scored.md
[DATE]_[SID]_Counter_Thesis.md
[DATE]_[SID]_Trade_Execution.md
[DATE]_[SID]_Master_Summary.md
[DATE]_[SID]_Portfolio_Log.md      ← SIMULATION + EXECUTION only
[DATE]_[SID]_PostMarket_Review.md
```

**Example:** a run started at 14:20:33 on 2026-06-25 produces:
```
FinalResearchoutput/research/2026-06-25/
  2026-06-25_142033_Market_Regime.md
  2026-06-25_142033_Daily_Brief.md
  2026-06-25_142033_Top5_Scored.md
  2026-06-25_142033_Counter_Thesis.md
  2026-06-25_142033_Trade_Execution.md
  2026-06-25_142033_Master_Summary.md
```

A second run at 16:05:11 on the same day adds `2026-06-25_160511_*.md` alongside — both sets are preserved.

**BACKTESTING mode** files go to `FinalResearchoutput/backtesting/[RUNDATE]/`:
```
[RUNDATE]_[SID]_Backtest_Summary.md      ← aggregate metrics + scope caveat
[RUNDATE]_[SID]_Backtest_TradeLog.md     ← per-day, per-trade decisions
[RUNDATE]_[SID]_Backtest_SignalReport.md ← signal accuracy by type and regime
```
State files appended (tagged BACKTEST, never mixed with SIMULATION/LIVE entries):
```
state/trade_history.json       ← SID-keyed blocks; prior runs preserved
state/signal_performance.json  ← accuracy data for this run
```

Legacy output from before the mode-subdirectory structure lives in `FinalResearchoutput/2026-06-20/` and `FinalResearchoutput/2026-06-22/` — do not use as templates.

---

## Mode Switching Protocol

| Transition | Safety Required |
|-----------|----------------|
| RESEARCH → SIMULATION | None |
| SIMULATION → RESEARCH | Warn if open paper positions exist |
| SIMULATION → EXECUTION | ≥10 simulation days tracked + win rate ≥45% + Robinhood MCP connected |
| EXECUTION → SIMULATION | Always allowed — emergency fallback |
| Any mode → BACKTESTING | None; always allowed |
| BACKTESTING → RESEARCH | None |
| BACKTESTING → SIMULATION | None (backtest does not affect portfolio_ledger.json) |

**Critical:** Backtest days in `state/trade_history.json` (tagged `source: "BACKTEST"`) do **NOT** count toward the ≥10 simulation days required for EXECUTION mode. Only forward paper-traded days in `state/portfolio_ledger.json` count.

---

## Data Sources & Tool Setup

- **Robinhood MCP** (`robinhood-trading`, HTTP, `https://agent.robinhood.com/mcp/trading`): live quotes, stop monitoring, post-market close prices, EXECUTION orders
  - Register project-scope via CLI: `claude mcp add robinhood-trading --transport http https://agent.robinhood.com/mcp/trading` — see `tool/mcp/robinhood/` for full docs.
  - To remove: `claude mcp remove robinhood-trading` | To verify: `claude mcp list`
  - Only the account you designate as "Agentic" (configured via `claude mcp add`) routes orders — see `tool/mcp/robinhood/README.md` for setup
  - **SHORT trades cannot be executed on Robinhood.** Use inverse ETFs or put options instead. See Step 8b in `trade_execution_thinking.md`.
  - Key tools by pipeline step:

| Step | Tool | Purpose |
|------|------|---------|
| Step 0.5 / F | `mcp__robinhood-trading__get_equity_quotes` | Live price for open positions + entry validation |
| Step 1 (tech agent) | `mcp__robinhood-trading__get_index_quotes` | SPX / NDX / QQQ / SPY live data |
| Step 1 (earnings agent) | `mcp__robinhood-trading__get_earnings_calendar` | Upcoming binary events |
| Step 1 (earnings agent) | `mcp__robinhood-trading__get_earnings_results` | Post-report actuals |
| Step 1 (thesis) | `mcp__robinhood-trading__get_equity_fundamentals` | P/E, revenue, margins |
| Step 3 | `mcp__robinhood-trading__search` | Instrument lookup by ticker |
| Step 5 (EXECUTION) | `mcp__robinhood-trading__review_equity_order` | Preview order before placing |
| Step 5 (EXECUTION) | `mcp__robinhood-trading__place_equity_order` | Submit order (human confirm first) |
| Step 5 (EXECUTION) | `mcp__robinhood-trading__cancel_equity_order` | Cancel pending order |
| Step 8 | `mcp__robinhood-trading__get_equity_historicals` | Official close prices |
| Step 8 | `mcp__robinhood-trading__get_portfolio` | Account equity + buying power |
| Step 8 | `mcp__robinhood-trading__get_realized_pnl` | Closed trade P&L |
| Verify auth | `mcp__robinhood-trading__get_accounts` | Confirm connection + account ID |
- **WebSearch**: news, earnings previews, analyst commentary, geopolitical events (fallback when Robinhood MCP not connected)
- **Gmail MCP** (connector UUID: `YOUR_GMAIL_CONNECTOR_UUID` — see setup in `tool/mcp/gmail/README.md`): daily email reports when `mode.email_report = true`
- **Tool directory**: `tool/` contains READMEs and config for every MCP server. Update `tool/mcp/<name>/config.json` after adding or changing any server.

---

## Known Limitations (critical for accurate behavior)

Do not over-represent these capabilities to the user:

- **Signal scores are qualitative, not backtested.** The BULL/BEAR/SIDEWAYS thresholds (±2.0) and agent weights (all 0.20) are arbitrary starting values, not empirically derived.
- **Robinhood MCP is the primary price source** (`get_equity_quotes`, `get_index_quotes`). WebSearch is the fallback only if the MCP is disconnected. Always prefer MCP quotes for Step F and portfolio review.
- **Intraday monitoring requires manual triggering.** There is no continuous automated monitoring. Stops are checked at session start (portfolio_review.md) and whenever the user runs intraday monitoring manually.
- **Learning weights do not auto-update.** The `rolling_20day_weight` fields in `state/performance_log.json` are conceptual — no code computes or applies them automatically. Every session starts with equal weights.
- **No options strategy module.** Binary events (earnings, FOMC) with gap risk are WATCH-ONLY. There is no mechanism to trade them with defined risk.
- **Robinhood does not support short selling.** SHORT trade cards generated in SIMULATION and RESEARCH have no direct EXECUTION equivalent. In EXECUTION mode, SHORT theses must be expressed via: (1) **inverse ETFs** (e.g., SQQQ/PSQ for Nasdaq shorts, SH for S&P 500), or (2) **put options** (requires enabling options on the Agentic account in the Robinhood app first). Step 8b in `trade_execution_thinking.md` blocks any direct SHORT order and requires explicit human selection of the replacement instrument.
- **Put options require explicit enablement.** Options are not automatically available on the Agentic account. Enable via the Robinhood app before attempting any options order in EXECUTION mode.
- **Minimum 10 simulation days before EXECUTION mode.** Less than that and there is insufficient data to validate any edge.
- **$100 simulation capital** produces psychologically insignificant P&L ($1-5/day). It tests mechanics, not behavior under real financial pressure.

---

## Workflow File Reference

```
workflows/
├── orchestrator.md            ← Entry point; reads mode; runs full pipeline
├── portfolio_review.md        ← Step 0.5; reviews open positions before new analysis
├── market_regime_detection.md ← Classifies BULL/BEAR/SIDEWAYS from 5-agent score
├── daily_market_brief.md      ← Macro signals + 10-15 company shortlist
├── top5_selection.md          ← Regime-adjusted Impact Score = Mag × Prob × Sens
├── event_analysis.md          ← Bull/bear/mispricing per company (runs before counter_thesis)
├── counter_thesis.md          ← Adversarial gate; Steps 0a/0b/A–G; APPROVE/MODIFY/REJECT
├── sector_exposure_control.md ← Step 4.5; sector caps (40% soft/50% hard); prevents concentration
├── trade_execution_thinking.md← Trade cards: entry/stop/target/R:R/EXECUTE/WATCH/REJECT
├── master_summary.md          ← 1-2 best trades; 5 sections; email content
├── intraday_monitoring.md     ← Regime-filtered signal scan; appends to day's files
├── post_market_review.md      ← Outcome tracking; regime accuracy; signal weight review
├── potfolio_tracker.md        ← P&L calculation; equity curve; stop check (filename typo is intentional — do not rename)
└── signal_agents/
    ├── macro_agent.md         ← Fed/yields/inflation/USD → score −5 to +5
    ├── earnings_agent.md      ← Earnings calendar/guidance/crowding → score + binary flags
    ├── tech_agent.md          ← Price/volume/VIX/breadth → score (use Robinhood MCP if connected)
    ├── sentiment_agent.md     ← Analyst activity/short interest/crowding → score
    └── geo_agent.md           ← Tariffs/geopolitical/commodities → score + ticker exposure map

readMe/
├── Solution_Document.md       ← Full system documentation: architecture, design decisions, limitations, roadmap, glossary
└── Docx/                      ← Document exports (currently empty)

resources/
└── watchlist.md               ← Active monitoring tickers + upcoming binary event calendar

tool/
├── README.md                  ← Master index: all MCP servers and built-in tools with status
├── mcp/
│   ├── robinhood/             ← Live quotes + order execution (✅ connected: robinhood-trading)
│   ├── gmail/                 ← Email reports (connected)
│   ├── google-drive/          ← Not yet active
│   └── vercel/                ← Not used in pipeline
└── builtin/
    ├── websearch/             ← Signal agents, price checks (fallback)
    └── webfetch/              ← Structured data from known URLs
```
