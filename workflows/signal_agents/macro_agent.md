# Signal Agent: Macro

## Objective

Identify macroeconomic signals that determine the BULL/BEAR/SIDEWAYS regime component.

Runs in parallel with 4 other signal agents. Output writes to `state/signal_cache.json` under `macro_agent`.

---

## Searches to Run

1. "Federal Reserve interest rates inflation outlook today [month year]"
2. "US Treasury yields 10-year 2-year spread today"
3. "CPI PCE inflation data latest 2026"
4. "Fed dot plot rate hike cut expectations 2026"
5. "USD index DXY dollar strength today"

---

## Signals to Score

| Signal | BULL (+1) | NEUTRAL (0) | BEAR (−1) |
|--------|-----------|-------------|-----------|
| Fed stance | Dovish (cutting/pausing) | Holding neutral | Hawkish (hiking/signaling hike) |
| 10yr yield | Falling | Stable | Rising sharply |
| Inflation trend | Declining toward 2% | Stable 2-3% | Rising above 3% |
| Yield curve | Steepening | Flat | Inverted / inverting more |
| USD | Weakening | Stable | Strengthening rapidly |

---

## Output Format

Write to `state/signal_cache.json` → `macro_agent`:

```json
{
  "status": "complete",
  "date": "[today]",
  "score": -1,
  "summary": "Fed hawkish (PCE 3.6%, dot plot hike signal). 10yr yield 4.46%. USD strong.",
  "signals": [
    {"name": "Fed stance", "value": -1, "evidence": "Warsh: 'unanimous commitment to price stability'; 9/18 back hike"},
    {"name": "10yr yield", "value": -1, "evidence": "4.46%; rising from 4.19% post-FOMC"},
    {"name": "Inflation trend", "value": -1, "evidence": "PCE 3.6% (target 2%); core 3.3%"},
    {"name": "Yield curve", "value": 0, "evidence": "Upward sloping; not inverted"},
    {"name": "USD", "value": -1, "evidence": "USD +1% on FOMC day; strong"}
  ]
}
```

---

## Rules

- Score is the sum of all 5 signal values (range: −5 to +5)
- If data is unavailable for a signal, score it 0 (NEUTRAL)
- Cite a specific source or data point for each signal
- Complete within 3-5 web searches
