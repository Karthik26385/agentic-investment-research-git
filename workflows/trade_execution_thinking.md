# Workflow: Trade Execution Thinking

## Objective

Convert research insights into **structured, risk-managed trade decisions**

---

## Step 1: Trade Qualification

Before considering execution, validate:

* Is there a **clear catalyst**?
* Is the impact **financially meaningful**?
* Is the opportunity **time-bound**?

If NO → Reject trade

---

## Step 2: Trade Type Identification

Classify the trade:

* Momentum (trend continuation)
* Mean Reversion (overreaction)
* Event-Driven (earnings, news, macro)
* Breakout (technical + volume)

---

## Step 3: Entry Strategy

Define **how to enter**:

* Immediate entry (if strong catalyst + early move)
* Pullback entry (wait for dip)
* Breakout entry (enter after resistance break)

Output:

* Preferred entry type
* Entry price range (approx)

---

## Step 4: Risk Management

Define **downside first**:

* What invalidates the thesis?
* Key support level

Set:

* Stop Loss (price or %)
* Max acceptable loss per trade

Rule:

* Never risk > 1–2% of capital per trade

---

## Step 5: Position Sizing

Determine allocation:

Factors:

* Confidence level
* Volatility
* Market conditions

Guideline:

* High conviction → Larger size
* Low conviction → Smaller size

Output:

* % of capital allocation

---

## Step 6: Target & Exit Plan

Define upside:

* Base case target
* Bull case target

Exit conditions:

* Target reached
* Thesis invalidated
* Time-based exit (if no movement)

---

## Step 7: Risk/Reward Check

Calculate:

* Risk = Entry – Stop Loss
* Reward = Target – Entry

Rule:

* Minimum Risk/Reward = 1:2
* Prefer 1:3+

If not met → Reject trade

---

## Step 8: Final Trade Decision

Output:

* Ticker
* Trade Type
* Entry Plan
* Stop Loss
* Target
* Position Size
* Risk/Reward Ratio

Final Call:

* EXECUTE / WATCH / REJECT

---

## Step 8b: SHORT Trade Gate (EXECUTION mode only)

Run this check immediately after Step 8 whenever `direction = SHORT` and `mode = EXECUTION`.

**Robinhood does not support short selling of individual equities.**

**Required action — do NOT convert silently:**

1. Block the direct short order.
2. Present the two valid alternatives:

   **Option A — Inverse ETF**
   - Map the short thesis to the closest inverse ETF:
     | Short target | Inverse ETF options |
     |-------------|-------------------|
     | Nasdaq / tech names | SQQQ (3×), QID (2×), PSQ (1×) |
     | S&P 500 | SPXS (3×), SDS (2×), SH (1×) |
     | Semiconductors | SOXS (3×), USD (2×) |
     | Individual sector short | Sector inverse ETF if available |
   - Warn: leveraged inverse ETFs (2×/3×) suffer decay in sideways markets — prefer 1× for holds > 2 days.
   - Recalculate entry/stop/target/R:R against the chosen inverse ETF price.

   **Option B — Put Option (requires options to be enabled on the Agentic account)**
   - Verify options are enabled in the Robinhood app before proceeding.
   - If not enabled → direct user to enable options on the Agentic account first.
   - If enabled → select put with 30–60 DTE, delta ~0.40–0.50 at target strike.
   - R:R: max loss = premium paid; reward = intrinsic value at expiry if thesis plays out.

3. Display to human:
   ```
   SHORT BLOCKED — Robinhood does not support short selling.
   Original thesis: SHORT [TICKER] at [ENTRY], stop [STOP], target [TARGET], R:R [X:Y]

   Alternative A (inverse ETF): [ETF] at ~$[PRICE] | Adjusted R:R: [X:Y]
   Alternative B (put option):  [OPTIONS_STATUS] — [details or enable prompt]

   Select A, B, or CANCEL before proceeding.
   ```
4. Await explicit human selection. Do NOT place any order without confirmation.

This gate applies in EXECUTION mode only. In RESEARCH and SIMULATION, SHORT trade cards are recorded as-is (paper trades; no order placed).

---

## Step 9: Execution Discipline Rules

* Do NOT chase price blindly
* Do NOT average down without new thesis
* Respect stop loss strictly
* Avoid overtrading

---

## Step 10: Post-Trade Tracking (Optional but Recommended)

Track:

* Entry vs exit
* Outcome (Win/Loss)
* What worked / failed

---

## Integration

This workflow runs AFTER:
`event_analysis.md`

And feeds into:
`master_summary.md` (optional trade section)

---

## Output Style

* Bullet points only
* No fluff
* Clear numeric levels

---

## Goal

Think like a **portfolio manager**, not just an analyst.

Convert:
Insight → Structured Trade → Controlled Risk

---
