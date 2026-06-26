# Signal Agent: Earnings

## Objective

Identify earnings-driven signals: upcoming binary events, recent guidance changes, estimate revisions.

Runs in parallel with 4 other signal agents. Output writes to `state/signal_cache.json` under `earnings_agent`.

---

## Searches to Run

1. "major earnings reports this week [date] [month year]"
2. "earnings guidance raised lowered [month year] large cap"
3. "analyst estimate revisions upgrades downgrades today"
4. "earnings calendar upcoming binary events week"
5. "earnings beats misses guidance [current month] 2026"

---

## Signals to Score

| Signal | BULL (+1) | NEUTRAL (0) | BEAR (−1) |
|--------|-----------|-------------|-----------|
| Recent earnings trend | Majority beating + raising | Mixed | Majority missing or lowering |
| Guidance revisions | Upward | Flat | Downward |
| Estimate revisions | Rising (upgrades dominant) | Balanced | Falling (downgrades dominant) |
| Upcoming binary risk | No major events this week | 1-2 medium events | High-stakes binary (e.g., mega-cap earnings) |
| Sector earnings trend | AI/Tech beating across board | Mixed | Key sector missing (e.g., semis miss) |

---

## Binary Events Tracking

For each upcoming earnings event in the next 5 days, record:

```json
{
  "ticker": "MU",
  "date": "2026-06-24",
  "event": "Q3 FY2026 earnings",
  "key_metric": "Gross Margin ~81%",
  "consensus_revenue": 34.66,
  "gap_risk": "HIGH",
  "regime_impact": "Micron beat = SIDEWAYS→BULL; miss = BEAR"
}
```

Flag HIGH gap_risk for any event where:
- Options implied move > 8%
- Wall Street unanimous (all buy = priced for perfection)
- Company guided conservatively vs analyst estimates

---

## Output Format

Write to `state/signal_cache.json` → `earnings_agent`:

```json
{
  "status": "complete",
  "date": "[today]",
  "score": 0,
  "summary": "Micron June 24 binary (81% GM critical). FedEx Q4 June 23. No broad earnings trend yet.",
  "signals": [...],
  "upcoming_binaries": [...]
}
```

---

## Rules

- Identify ALL earnings in the next 5 trading days
- Flag any with gap_risk = HIGH (these go to counter-thesis Step G automatically)
- Score binary presence as BEAR if 2+ high-stakes events this week (uncertainty = risk)
