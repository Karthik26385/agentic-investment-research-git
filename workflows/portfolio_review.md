# Workflow: Portfolio Review (Step 0)

## Objective

Before scanning for new opportunities, review every open position in `state/portfolio_ledger.json` and generate an action signal: **HOLD / ADD / REDUCE / EXIT**.

This runs at the START of every orchestrator cycle — before signal agents, before new theses.

---

## When to Run

* First step of every `orchestrator.md` execution
* Also triggered by intraday_monitoring when price moves > ±3% on an open position

---

## Input

Read `state/portfolio_ledger.json` → `positions` array.
If positions array is empty → skip to Step 1 (signal agents).

**Sector exposure reconciliation (runs before any position review):**
Rebuild `state/sector_exposure.json` from scratch using only `positions` (open trades) in portfolio_ledger.json.
For each open position: look up its sector in `sector_exposure.json` → `_sector_map`, compute `capital_deployed = cost_basis`, compute `exposure_pct = cost_basis / account.current_capital × 100`.
Compare rebuilt values against the stored `current_exposure` in sector_exposure.json.
```
If any sector's stored exposure_pct differs from the rebuilt value by > 2% →
  Overwrite sector_exposure.json with the rebuilt values and log:
  "SECTOR EXPOSURE RECONCILED: stored values were out of sync with portfolio_ledger.json"
```
This prevents stale or manually-edited sector caps from silently bypassing concentration limits.

---

## For Each Open Position — Run These Steps:

### Step A: Get Current Price

Search: `"[TICKER] stock price today"` OR use Robinhood MCP `get_quote("[TICKER]")`.

Record:
* Current price
* % change from entry
* Distance to stop (%)
* Distance to target (%)

---

### Step B: Stop & Target Check

**Stop hit?**
```
If current_price <= stop_price → FLAG: EXIT (stop triggered)
Action: Close position immediately. Record: exit_reason = "stop_hit"
```

**Target hit?**
```
If current_price >= target_price → FLAG: TAKE PROFIT
Action: Sell 50% at target. Raise stop to entry price (free trade). Update target to bull_case.
```

**Near stop (within 3%)?**
```
If (current_price - stop_price) / entry_price < 0.03 → FLAG: STOP WARNING
Action: Re-evaluate thesis urgently (skip to Step C immediately)
```

---

### Step C: Thesis Re-Evaluation

Search: `"[TICKER] news today [date]"` to find any developments since entry.

Ask:
1. Is the original catalyst still active? (e.g., WDC: was the arb fully absorbed?)
2. Does today's news strengthen or weaken the thesis?
3. Has the invalidation condition been triggered?

Re-run `counter_thesis.md` on the existing position:
* Steps A-G still apply
* If counter-thesis is now STRONG → FLAG: EXIT (thesis broken)
* If counter-thesis unchanged → no action from this step

---

### Step D: Regime Compatibility Check

Read current regime from `state/regime_state.json`.

Compare to regime at entry:
```
Regime improved (e.g., SIDEWAYS → BULL) + position is LONG → FLAG: ADD
Regime deteriorated (e.g., SIDEWAYS → BEAR) + position is LONG → FLAG: REDUCE (cut size 50%)
Regime unchanged → FLAG: HOLD (no regime-based action)
```

**Regime-based sizing adjustment:**
* BULL: Allow position to grow to 1.5× original size
* SIDEWAYS: Maintain original size
* BEAR: Reduce to 0.5× original size or EXIT if LONG

---

### Step E: Profit Protection

**Trailing stop logic (activate when unrealized gain > 7%):**
```
If unrealized_pct >= 7% AND no stop raise yet:
    Raise stop to entry_price (break-even — free trade)

If unrealized_pct >= 12%:
    Raise stop to entry + 5% (lock in partial gain)

If unrealized_pct >= target_pct × 0.5 (halfway to target):
    Sell 25% of position, trail stop on remainder
```

---

### Step F: Output Decision

Produce one action per position:

```
HOLD     → No action required. Continue monitoring.
ADD      → Regime improved + thesis strengthened → buy more (up to regime cap)
REDUCE   → Regime deteriorated or thesis weakening → cut size 50%
EXIT     → Stop hit, thesis broken, or regime incompatible → close position
TAKE_PROFIT → Target hit → sell 50%, trail remainder
STOP_WARNING → Price within 3% of stop → watch closely
```

---

## Output Format

Write to `state/portfolio_ledger.json` → each position gets an `action` field:

```json
{
  "ticker": "NVDA",
  "action": "HOLD",
  "action_reason": "Thesis intact; regime unchanged SIDEWAYS; price $243 (7.4% above stop $226)",
  "price_checked": 243.00,
  "unrealized_pct": 0.83,
  "stop_distance_pct": 7.4,
  "target_distance_pct": 12.4,
  "profit_protection": "Stop raise to $241 (break-even) triggers at +7% = $258"
}
```

---

## Integration in Orchestrator

```
orchestrator.md
    │
    ▼
STEP 0 — portfolio_review.md  ← THIS WORKFLOW
    │ (executes exits/additions before any new analysis)
    ▼
STEP 1 — 5 Parallel Signal Agents
    │
    ▼
... (rest of pipeline)
```

---

## Rules

* NEVER skip this step if positions are open
* EXIT actions execute immediately — do not wait for end of day
* ADD actions count toward the daily max_daily_trades limit
* REDUCE and EXIT free up cash for new positions
* All actions written to portfolio_ledger.json with timestamp

---

## Goal

Ensure the portfolio is actively managed, not just passively monitored.
Every day the system must answer: **Should I still own this?**

---
