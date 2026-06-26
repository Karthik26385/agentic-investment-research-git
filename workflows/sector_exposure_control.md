# Workflow: Sector Exposure & Capital Allocation Control

## Objective

Prevent overexposure to a single sector while allowing **exception-based concentration when signals are extremely strong**

---

## When to Run

* After `counter_thesis_validation.md`
* Before `trade_execution_thinking.md`
* For ALL approved trades before execution

---

## Step 1: Map Each Trade to Sector

Assign sector tag:

Examples:

* NVDA → Technology (Semiconductors)
* AMD → Technology (Semiconductors)
* AAPL → Technology (Consumer Devices)
* TSLA → Consumer Discretionary (EV)
* XOM → Energy

---

## Step 2: Calculate Current Exposure

From portfolio:

* Capital allocated per sector
* Number of positions per sector

---

## Step 3: Define Risk Limits

### Default Limits:

* Max Sector Exposure: **40% of total capital**
* Max Positions per Sector: **2 trades**

---

## Step 4: Evaluate New Trade Impact

For each new trade:

Check:

* Will this exceed 40% sector exposure?
* Will this create >2 positions in same sector?

---

## Step 5: Decision Logic

### Case A: Within Limits

* ✅ APPROVE TRADE

---

### Case B: Exceeds Limits

Check Signal Strength:

#### Strong Signal (High Conviction)

Criteria:

* Passed counter-thesis with WEAK risk
* Clear catalyst
* Favorable R:R ≥ 1:3

👉 Action:

* ⚠️ ALLOW but:

  * Reduce size by 30–50%
  * Flag as “Concentrated Bet”

---

#### Medium Signal

* ⚠️ REDUCE POSITION SIZE
* OR delay entry

---

#### Weak Signal

* ❌ REJECT TRADE

---

## Step 6: Capital Allocation Adjustment

If multiple trades in same sector:

* Rank trades by:

  * Signal strength
  * R:R ratio
  * Counter-thesis score

👉 Allocate capital to TOP 1–2 only

---

## Step 7: Output Format

### Sector Exposure Summary

* Sector: Technology

  * Current Exposure: 35%
  * After Trade: 52%

---

### Decision

* Status: APPROVE / REDUCE / REJECT
* Reason: Sector overexposure
* Action Taken: Size reduced / trade skipped

---

## Step 8: Update Portfolio State

Add:

* sector_exposure.json

Example:

```json id="g5u5vv"
{
  "Technology": 38,
  "Energy": 12,
  "Healthcare": 10
}
```

---

## Integration

```text id="mtw4ab"
counter_thesis_validation.md
        ↓
sector_exposure_control.md   ← NEW
        ↓
trade_execution_thinking.md
```

---

## Rules

* Never exceed 50% sector exposure (hard cap)
* Prefer diversification unless conviction is extremely high
* Do not take 3 correlated trades in same sector

---

## Goal

Avoid:

* Hidden correlation risk
* Sector concentration losses

Enable:

* Controlled conviction-based bets

---

## Final Principle

> “Diversify by default. Concentrate only with proof.”

---
