# Workflow: Intraday Market Monitoring & Signal Execution Loop

## Objective

Continuously monitor markets during trading hours to:

* Identify new opportunities
* Convert them into structured trades
* Append insights into daily outputs

---

## When to Run

* Start: Market Open (9:30 AM ET)
* Frequency: Every 1–2 hours
* End: Market Close (4:00 PM ET)

---

## Input Sources

* Price movements (top gainers/losers)
* Volume spikes
* Breaking news
* Sector momentum
* Macro updates during the day

---

## Step 1: Intraday Signal Scan

Identify candidates based on:

* Price move > ±3% intraday
* Unusual volume spike
* Breaking news or headlines
* Sector-wide moves (e.g., semiconductors rally)
* Gap up / gap down stocks

---

## Step 2: Quick Signal Validation

For each candidate:

Check:

* Is there a **clear catalyst**?
* Is movement **significant vs normal volatility**?
* Is it **not already captured in pre-market analysis**?

If NO → Ignore
If YES → Proceed

---

## Step 3: Categorize Opportunity

Tag as:

* Momentum continuation
* Reversal (overreaction)
* News-driven event
* Sector rotation

---

## Step 4: Mini Event Analysis (Lightweight)

Quickly evaluate:

* What changed?
* Is this likely to impact financials or sentiment?
* Is move justified or overextended?

---

## Step 5: Convert to Trade Execution Thinking

Pass qualified signals to:

→ `trade_execution_thinking.md`

Define:

* Entry strategy (immediate / pullback / breakout)
* Stop loss
* Target
* Position sizing
* Risk/Reward

---

## Step 6: Trade Qualification Filter

Only proceed if:

* Risk/Reward ≥ 1:2
* Clear catalyst
* Not duplicate of existing trade

Else → Reject

---

## Step 7: Append to Daily Master Summary

Update:

### Section: Intraday Updates (NEW)

Add:

* Ticker
* Catalyst
* Direction
* Short note

---

## Step 8: Append to Trade Execution Section

If trade qualifies:

Add under:

### Intraday Trade Additions

Include:

* Full trade structure
* Timestamp (important)

---

## Step 9: Avoid Overtrading

Rules:

* Max 1 additional trade per cycle
* Max 2 intraday trades per day
* Do NOT override existing trades unless thesis invalidated

---

## Step 10: Loop Continuation

Wait 1–2 hours → Repeat from Step 1

---

## Step 11: End of Day Sync

Before market close:

* Consolidate all intraday updates
* Ensure Master Summary reflects:

  * Final Top 5
  * All valid trades (max 2–3 total)

---

## Integration with System

Pre-Market:

* `daily_market_brief.md`
* `top5_selection.md`
* `event_analysis.md`

Intraday:

* `intraday_monitoring.md` (this workflow)
* `trade_execution_thinking.md`

Post-Market:

* `post_market_review.md`

---

## Output Updates

### Master File:

Append to: <DATE>_Master_Summary.md

### Trade File:

Append to: <DATE>_Trade_Execution.md (optional separate file)

---

## Rules

* Do NOT chase random moves
* Do NOT react without catalyst
* Focus on quality over quantity
* Maintain discipline across cycles

---

## Goal

Build a **real-time adaptive investment system**

Combine:
Pre-market conviction

* Intraday signal detection
* Structured execution
* Post-market learning

---

## Final Note

You are NOT trading real money.

Focus:

* Signal quality
* Decision accuracy
* Process discipline

NOT:

* Over-optimization
* Overtrading

---
