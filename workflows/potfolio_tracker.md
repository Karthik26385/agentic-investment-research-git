# Workflow: Portfolio Tracker & Capital Management

## Objective

Track daily portfolio performance, capital allocation, and trade outcomes to simulate a **real investment fund**

---

## Initial Setup

* Starting Capital: $100
* Max Risk per Trade: 2% of capital
* Max Active Trades: 2
* Trade Type: Simulated (no real execution)

---

## When to Run

* After each trade execution
* After market close (mandatory daily update)

---

## Step 1: Initialize Daily Portfolio State

Capture:

* Date
* Starting Capital
* Active Trades (if any carried forward)

---

## Step 2: Log Trade Entry

For each trade:

Record:

* Ticker
* Trade Type (Long/Short)
* Entry Price
* Stop Loss
* Target Price
* Position Size (shares)
* Capital Allocated ($)
* Risk Amount ($)

---

## Step 3: Track Intraday Status (Optional)

Update:

* Current price
* Unrealized PnL
* Status: Open / Near Target / Near Stop

---

## Step 4: Log Trade Exit

When trade closes:

Capture:

* Exit Price
* Exit Reason:

  * Target Hit
  * Stop Loss Hit
  * Manual Close
  * End-of-Day Close

---

## Step 5: Calculate Trade PnL

### For Long:

PnL = (Exit - Entry) × Shares

### For Short:

PnL = (Entry - Exit) × Shares

---

## Step 6: Update Capital

New Capital:

```text
New Capital = Previous Capital + Total Daily PnL
```

---

## Step 7: Daily Performance Metrics

Calculate:

* Daily PnL ($ and %)
* Total Capital
* Number of Trades
* Wins / Losses
* Win Rate

---

## Step 8: Risk Metrics

Track:

* Max Drawdown (from peak capital)
* Avg Gain per Trade
* Avg Loss per Trade
* Risk/Reward realized

---

## Step 9: Portfolio Summary Entry

Append to daily log:

### 📊 Daily Summary

* Starting Capital:
* Ending Capital:
* Daily Return:
* Trades Taken:
* Wins / Losses:
* Key Trades:

---

## Step 10: Maintain Trade Log History

Maintain cumulative table:

| Date | Ticker | Type | Entry | Exit | PnL | Result |
| ---- | ------ | ---- | ----- | ---- | --- | ------ |

---

## Step 11: Equity Curve Tracking

Track:

* Daily capital progression

Example:

```text
Day 1 → $100  
Day 2 → $104  
Day 3 → $102  
Day 4 → $107  
```

---

## Step 12: Weekly Review (Every 5 Days)

Summarize:

* Total Return %
* Win Rate
* Best Trade
* Worst Trade
* Key Learnings

---

## File Structure

### Daily File:

```text
<DATE>_Portfolio_Log.md
```

### Location:

FinalResearchoutput/YYYY-MM-DD/

---

## Integration

* Inputs from:

  * `trade_execution_thinking.md`
  * `intraday_monitoring.md`

* Feeds into:

  * `post_market_review.md`

---

## Rules

* Always update capital after each closed trade
* Never exceed risk per trade (2%)
* Do not retroactively modify trades
* Maintain full history (no deletion)

---

## Goal

Simulate a **real portfolio manager mindset**

Track:
Trades → PnL → Capital → Risk → Growth

---

## Final Note

Focus is NOT:

* Maximum profit

Focus IS:

* Consistent growth
* Controlled risk
* Process discipline

---
