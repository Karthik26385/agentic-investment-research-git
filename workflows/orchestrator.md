# Workflow: Master Orchestrator

## Objective

Coordinate the full autonomous pipeline by reading `state/mode.json` and running every step in the correct order with the correct mode-specific behavior.

---

## Step 0: Read Active Mode + Generate Session ID

Read `state/mode.json` → get `active_mode`.

Extract from the active mode config:
- `output_dir` → where to write daily files
- `portfolio_tracking` → whether to update portfolio_ledger.json
- `execution` → none / paper / real
- `position_size_cap_pct` → max position size for this mode
- `max_daily_trades` → cap on trades per day
- `human_confirmation` → block before execution if true

**Generate SESSION_ID** = current wall-clock time in `HHMMSS` format (e.g. `142033` for 14:20:33 ET).
This ID is prepended to every output file name produced in Step 6 and must stay constant for the entire pipeline run.

Print at start of every run:
```
MODE:               [RESEARCH | SIMULATION | EXECUTION | BACKTESTING]
Output:             [output_dir]/[YYYY-MM-DD]/
Portfolio tracking: [ON/OFF]
Max trade size:     [X%]
SESSION_ID:         [SID]
```

**If active_mode == BACKTESTING:**
```
1. Read lookback_days from the trigger command
     "Start BACKTESTING 10 Days" → N = 10
     "Start BACKTESTING"         → N = mode.default_lookback_days (10)
2. Compute:
     sim_end   = most recent completed trading day (yesterday, or last close if today is weekend/holiday)
     sim_start = sim_end minus N trading days (skip weekends and market holidays)
3. Print extended header:
     MODE:               BACKTESTING
     Output:             FinalResearchoutput/backtesting/[RUNDATE]/
     Portfolio tracking: OFF
     Lookback:           N trading days ([sim_start] to [sim_end])
     SESSION_ID:         [SID]
4. Route to backtesting_engine.md with (sim_start, sim_end, N, SID, mode config)
5. Do NOT run Steps 0.5 through 8 — the live pipeline is bypassed entirely.
6. After backtesting_engine.md completes → stop. Return summary to user.
```

**All other modes** proceed to Step 0.5 and the standard 9-step pipeline as normal.

---

## Step 0.5: Portfolio Review (runs if open positions exist)

Read `state/portfolio_ledger.json` → `positions` array.

If positions array has entries → execute `portfolio_review.md` for each open position:
- Get current price (Robinhood MCP or WebSearch)
- Check stop / target
- Re-evaluate thesis with today's news
- Check regime compatibility
- Apply profit protection (trailing stop)
- Output: HOLD / ADD / REDUCE / EXIT / TAKE_PROFIT for each

Write action results back to `state/portfolio_ledger.json` → each position's `action` field.

**Execute any EXIT or TAKE_PROFIT actions immediately** before proceeding to new analysis.
ADD actions are noted but counted against `max_daily_trades` limit for the day.

If positions array is empty → skip this step.

---

## Step 1: Spawn 5 Parallel Signal Agents

**Before spawning — stale cache check:**
Read `state/signal_cache.json` → check `last_updated` field.
```
If last_updated == today's date → cache is fresh; agents may use cached scores as a baseline
If last_updated != today's date → cache is STALE; invalidate all scores; all 5 agents MUST run fresh
```
Do not carry forward signal scores from a previous session. A stale cache driving today's regime
classification is a silent data integrity failure.

Launch simultaneously:
- `signal_agents/macro_agent.md`
- `signal_agents/earnings_agent.md`
- `signal_agents/tech_agent.md`
- `signal_agents/sentiment_agent.md`
- `signal_agents/geo_agent.md`

Each agent writes its output to `state/signal_cache.json` under its agent key.
Each agent must also write `last_updated: [today's date]` to the cache upon completion.

Wait for all 5 to complete before proceeding.

---

## Step 2: Run Regime Engine

Read all 5 agent outputs from `state/signal_cache.json`.

Compute weighted aggregate score:
```
regime_score = Σ (agent_weight × agent_score)
```

Weights from `state/regime_state.json` → `signal_weights`.

Classify:
- score > 2.0  → BULL  (position cap: mode × 1.5)
- score -2 to 2 → SIDEWAYS (position cap: mode × 1.0)
- score < -2.0  → BEAR  (position cap: mode × 0.5)

Write result to `state/regime_state.json` → current + append to history.

---

## Step 3: Run Thesis Builder

Execute `daily_market_brief.md` then `top5_selection.md` using regime from Step 2.

Regime adjustment applied during scoring:
- SIDEWAYS/BEAR: Momentum trades −1 Sensitivity
- SIDEWAYS/BEAR: Mean Reversion trades +1 Sensitivity

Output: Top 5 thesis candidates → write to `state/thesis_queue.json` with status = "PENDING".

---

## Step 4: Run Counter-Thesis Agent (per candidate)

For each PENDING thesis in `state/thesis_queue.json`:

Execute `counter_thesis.md` (which reads mode from `state/mode.json` for Step 0).

Update each thesis in `state/thesis_queue.json`:
- status: APPROVED / WATCH / REJECTED
- counter_thesis_rating: WEAK / MEDIUM / STRONG
- counter_thesis_flags: count

---

## Step 4.5: Sector Exposure Control

Execute `sector_exposure_control.md` for all APPROVED theses from Step 4.

For each approved trade:
1. Map ticker to sector (e.g., NVDA → Technology/Semiconductors)
2. Read current exposure from `state/sector_exposure.json`
3. Check: would this trade exceed 40% sector cap or 2 positions per sector?
4. Decision:
   - Within limits → APPROVE (pass to Step 5)
   - Exceeds limit + WEAK CT + R:R ≥1:3 → ALLOW as "Concentrated Bet" (reduce size 30-50%)
   - Exceeds limit + MEDIUM CT → REDUCE or DELAY
   - Exceeds limit + WEAK CT + R:R <1:3 → REJECT
   - Hard cap: NEVER exceed 50% in any single sector

Update `state/sector_exposure.json` with new exposure after approved trades.

---

## Step 5: Run Execution Engine

Execute `trade_execution_thinking.md` for all APPROVED theses from Step 4.5.

Apply mode constraints:
- position_size = min(thesis_size, mode.position_size_cap_pct)
- If trades today already = max_daily_trades → mark remaining as WATCH
- R:R < 1:2 → REJECT regardless of mode

Mode-specific execution:
- **RESEARCH**: Write trade cards to markdown only. No ledger update.
- **SIMULATION**: Write trade cards + update `state/portfolio_ledger.json` (paper).
- **EXECUTION**: Write trade cards → show to human → await confirmation → execute via Robinhood MCP.

**EXECUTION-only dollar cap gate (runs before every order):**
Read `mode.json` → `max_order_usd`.
```
order_value = entry_price × planned_shares
If order_value > max_order_usd → BLOCK order
  Show: "ORDER BLOCKED: ${order_value} exceeds max_order_usd cap of ${max_order_usd}.
         Reduce size or raise max_order_usd in mode.json deliberately."
  Do NOT place the order. Do NOT reduce shares silently to fit. Require explicit human action.
```
This cap is absolute and applies even if position_size_cap_pct calculations permit the size.

---

## Step 6: Write Daily Output Files

Output directory = `[mode.output_dir]/[YYYY-MM-DD]/`

Use the SESSION_ID generated in Step 0. File pattern: `[DATE]_[SID]_[FileName].md`

Files written:
- `[DATE]_[SID]_Market_Regime.md`
- `[DATE]_[SID]_Daily_Brief.md`
- `[DATE]_[SID]_Top5_Scored.md`
- `[DATE]_[SID]_Counter_Thesis.md`
- `[DATE]_[SID]_Trade_Execution.md`
- `[DATE]_[SID]_Master_Summary.md`

In SIMULATION and EXECUTION modes, also write:
- `[DATE]_[SID]_Portfolio_Review.md` (from `portfolio_review.md`, if open positions exist)
- `[DATE]_[SID]_Portfolio_Log.md` (from `potfolio_tracker.md`)

Do NOT delete existing files in the date folder. Each run has a unique SID; prior runs are preserved.

---

## Step 7: Send Email Report (if mode.email_report = true)

Execute `master_summary.md` → format for email → send via Gmail MCP.

Subject: `[MODE] Daily Brief — [DATE] | [REGIME] | [SENTIMENT]`

In SIMULATION: include portfolio P&L section.
In EXECUTION: include real P&L + open positions.

---

## Step 8: Post-Market Update (runs at 4 PM ET)

Execute `post_market_review.md`:
1. Fetch actual close prices (Robinhood MCP or WebSearch)
2. Update `state/portfolio_ledger.json` — mark stops hit, targets hit, open positions
3. Update `state/performance_log.json` — signal accuracy, regime accuracy, trade outcomes
4. Adjust signal weights in `state/regime_state.json` based on rolling accuracy
5. Update `.claude/memory/` with key learnings

---

## Mode Safety Gates (EXECUTION only)

Before any real order:
1. `counter_thesis_rating` must NOT be STRONG
2. `regime` must NOT be BEAR if trade is LONG
3. Portfolio `max_drawdown_pct` must be < 5% (if >= 5%, halt new trades)
4. `human_confirmation` = true → display full trade card, await explicit "CONFIRM" response

---

## Integration Map

```
orchestrator.md
    │
    ├── Reads: state/mode.json
    ├── Reads/Writes: state/signal_cache.json
    ├── Reads/Writes: state/regime_state.json
    ├── Reads/Writes: state/thesis_queue.json
    ├── Reads/Writes: state/portfolio_ledger.json
    └── Reads/Writes: state/performance_log.json

Calls (in order):
    signal_agents/* → regime engine → daily_market_brief.md →
    top5_selection.md → counter_thesis.md → trade_execution_thinking.md →
    potfolio_tracker.md → master_summary.md → post_market_review.md
```

---

## Trigger

This workflow runs when the user says any of the following, or a scheduled cloud agent fires:

| Command | Action |
|---------|--------|
| `start` | Run full pipeline in current active mode |
| `start RESEARCH` | Switch to RESEARCH → run full pipeline |
| `start SIMULATION` | Switch to SIMULATION → run full pipeline |
| `SIMULATION - Run` | Same as "start SIMULATION" |
| `switch to EXECUTION` | Validate safety gates → switch |
| `Start BACKTESTING N Days` | Switch to BACKTESTING → run backtesting_engine.md for last N trading days |
| `Start BACKTESTING` | Switch to BACKTESTING → run with default 10 days |

First action: always read `state/mode.json`.

If `active_mode == BACKTESTING`:
→ Run `backtesting_engine.md` (historical simulation loop) — skip Steps 0.5–8.

All other modes:
→ Run the standard 9-step pipeline (Steps 0.5 through 8).
