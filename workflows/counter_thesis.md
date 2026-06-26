# Workflow: Counter-Thesis (Red Team) Validation

## Objective

For every trade candidate, actively search for **reasons the trade idea is wrong, incomplete, or dangerous** before approval.

This step is **mandatory before execution**.

---

## When to Run

* After `event_analysis.md`
* Before `trade_execution_thinking.md`
* For every BUY / SELL / HOLD candidate that becomes a trade

---

## Step 0a: Read Active Mode
Read `state/mode.json` → get `active_mode` and `position_size_cap_pct`.
- RESEARCH: analysis output only; no ledger update
- SIMULATION: approved trades update state/portfolio_ledger.json (paper)
- EXECUTION: approved trades require human confirmation before Robinhood order

Also read `state/signal_cache.json` → `sentiment_agent.crowded_positions` to pre-flag crowded setups before analysis begins.

## Step 0b: Price Reality Check (mandatory before analysis)
Search "[TICKER] current stock price" or use Robinhood MCP `get_quote("[TICKER]")` — verify against assumed entry.
If >10% off → STOP. Correct price. Restart from Step 1.

---

## Step 1: Restate the Trade Thesis

Clearly define:

* Ticker
* Direction (Long / Short)
* Catalyst
* Expected outcome

Example:

> “Long NVDA due to AI demand acceleration and strong earnings momentum”

---

## Step 2: Counter-Thesis Search (CRITICAL)

Actively search for opposing evidence:

### A. Macro Risks

* Interest rates rising?
* Sector headwinds?
* Liquidity tightening?

---

### B. Company-Specific Risks

* Earnings already priced in?
* Margin pressure?
* Guidance weakness?
* Competitive threats?

---

### C. Market Structure Risks

* Overcrowded trade?
* Already extended price move?
* Low risk/reward entry?

---

### D. Sentiment Risks

* Retail overhype?
* Analyst upgrades already priced?
* “Buy the rumor, sell the news” risk?

---

### E. Timing Risks

* Late entry in trend?
* Event already occurred?
* No near-term catalyst?

### F. Price Validity Check
* Search for current stock price. Compare against assumed entry range. If stock is >10% away from assumed entry, halt and recalibrate. Price errors invalidate the entire trade card.

### G. Binary Event Gap Protocol:
* If the trade thesis depends on a binary event (earnings, FOMC, product launch), calculate R:R assuming an 8-12% gap in either direction. If gap-adjusted R:R falls below 1:2, change recommendation from EXECUTE to WATCH-ONLY.
---

## Step 3: Bear Case Construction

Write a structured **Bear Case Thesis**:

* “This trade fails if…”
* Key invalidation signals
* Most likely failure scenario

---

## Step 4: Risk Severity Scoring

Rate counter-thesis strength:

* Weak (minor concerns)
* Medium (valid risks exist)
* Strong (trade likely flawed)

---

## Step 5: Decision Gate

### If Counter-Thesis is STRONG:

* ❌ REJECT TRADE

### If MEDIUM:

* ⚠️ REDUCE SIZE / WAIT / MODIFY ENTRY

### If WEAK:

* ✅ APPROVE TRADE

---

## Step 6: Adjust Trade Structure (If Approved)

Modify:

* Entry timing (delay / wait for pullback)
* Position size (reduce exposure)
* Stop loss (tighten if needed)
* Target (adjust expectations)

---

## Step 7: Output Format

### Trade Review

* Original Thesis:
* Counter-Thesis:
* Bear Case:
* Risk Rating: Weak / Medium / Strong

### Final Decision:

* APPROVE / MODIFY / REJECT

---

## Integration Points

This workflow sits between:

```text id="x7kqpl"
event_analysis.md
        ↓
counter_thesis_validation.md   ← HERE
        ↓
trade_execution_thinking.md
```

---

## Rules

* Never skip this step for active trades
* Must include at least 2 counter arguments
* Do NOT justify trade first—challenge it first
* Assume the trade is WRONG until proven otherwise

---

## Goal

Eliminate:

* Overconfidence bias
* Narrative bias
* Momentum chasing

---

## Outcome

A strong system will:

* Reject weak trades early
* Reduce false positives
* Improve win quality (not just win rate)

---

## Final Principle

> “A good trade is not one that looks right —
> it is one that survives being attacked.”

---
