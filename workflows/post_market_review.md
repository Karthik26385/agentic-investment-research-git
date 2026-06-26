# Workflow: Post-Market Validation & Performance Review

## Objective

Evaluate how all **daily recommendations and trades performed after market close**, and build a feedback loop for continuous improvement.

---

## When to Run

* After market close (4 PM ET)
* Once per day

---

## Input

* Daily Master Summary file
* Trade Execution details (if available)

---

## Step 1: Track Market Outcome

For each of the Top 5 companies:

Capture:

* Opening price
* Closing price
* % change
* Intraday high/low (optional)

---

## Step 2: Validate Recommendations

For each company:

### If Recommendation = BUY

* Did stock go UP?
* Strength of move (% gain)

### If Recommendation = SELL

* Did stock go DOWN?

### If Recommendation = HOLD

* Did stock stay neutral?

---

### Output:

* Correct / Incorrect / Neutral

---

## Step 3: Trade Performance (For Executed Trades Simulation)

For each trade:

### Entry Simulation

* Assume entry at defined entry range midpoint

### Check Outcomes:

* Did price hit Target?
* Did price hit Stop Loss?
* Did neither happen?

---

### Classification:

* WIN → Target hit
* LOSS → Stop hit
* OPEN → Neither hit

---

## Step 4: Performance Metrics

Calculate:

* Total trades: #

* Wins: #

* Losses: #

* Win Rate: %

* Avg Gain (winning trades)

* Avg Loss (losing trades)

* Risk/Reward realized

---

## Step 5: Insight Analysis

Identify patterns:

### What Worked:

* Correct event types (earnings, macro, etc.)
* Accurate sectors
* Good timing

### What Failed:

* Weak catalysts
* Overreaction misjudged
* Poor entry timing

---

## Step 6: Mistake Categorization

Tag errors:

* Signal Error → wrong event selection
* Analysis Error → wrong interpretation
* Execution Error → bad entry/exit
* Market Noise → unpredictable movement

---

## Step 7: Learning Summary

Write 3–5 bullet insights:

* What to repeat
* What to avoid
* What to refine in workflows

---

## Step 8: Feedback Loop

Write all learnings, corrections, and pattern observations to the daily `[DATE]_PostMarket_Review.md` output file only.

**DO NOT directly edit any workflow markdown file** (orchestrator.md, top5_selection.md, event_analysis.md, trade_execution_thinking.md, counter_thesis.md, or any signal agent). Automated self-modification of workflow instructions based on daily P&L is a security and reliability risk — workflow changes require explicit human review and a deliberate manual edit.

If this session's results reveal a systematic error in a workflow, append a clearly labelled section to the PostMarket output:

```
## Workflow Change Recommendation
File:   [workflow filename]
Issue:  [what went wrong]
Proposed change: [specific text to add/remove/replace]
Action required: Human review and manual edit before next session
```

---

## Output File

Save as: <DATE>_PostMarket_Review.md

Location:
FinalResearchoutput/YYYY-MM-DD/

---

## Output Structure

### Section 1: Market Results

* Summary of stock movements

### Section 2: Recommendation Accuracy

* Correct vs Incorrect

### Section 3: Trade Results (Simulated)

* Win / Loss / Open

### Section 4: Metrics

* Win rate
* Avg gain/loss

### Section 5: Key Learnings

* Bullet insights

---

## Rules

* Be brutally honest (no bias)
* No justification of wrong calls
* Focus on improvement, not perfection

---

## Goal

Create a **self-improving investment system**

Convert:
Daily Ideas → Measured Outcomes → Better Future Decisions

---
