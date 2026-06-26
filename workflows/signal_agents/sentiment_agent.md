# Signal Agent: Sentiment

## Objective

Identify sentiment-driven signals: analyst activity, short interest, positioning, retail flows, crowding.

Runs in parallel with 4 other signal agents. Output writes to `state/signal_cache.json` under `sentiment_agent`.

---

## Searches to Run

1. "analyst upgrades downgrades today [date] large cap stocks"
2. "short interest changes this week [month year]"
3. "institutional investor positioning sentiment [month year]"
4. "retail investor flows meme stocks today"
5. "put call ratio options market sentiment today"

---

## Signals to Score

| Signal | BULL (+1) | NEUTRAL (0) | BEAR (−1) |
|--------|-----------|-------------|-----------|
| Analyst revisions | Upgrades dominating | Balanced | Downgrades dominating |
| Short interest trend | Declining (shorts covering) | Stable | Rising (shorts adding) |
| Institutional positioning | Adding to longs | Neutral | Reducing / rotating out |
| Options sentiment (P/C ratio) | Put/call < 0.8 (bullish) | 0.8-1.0 | Put/call > 1.0 (bearish) |
| Crowding risk | Few consensus longs | Mixed | Heavily crowded trades |

---

## Crowding Detection

Flag a trade as CROWDED if:
- All analysts are on the same side (e.g., 27/27 Strong Buy on MU)
- Short interest is near 0 (no natural buyers left)
- Stock has run > 50% YTD before the catalyst

Crowded long: REDUCE position size by 50% regardless of R:R
Crowded short: only enter if confirmed negative catalyst exists

---

## Output Format

```json
{
  "status": "complete",
  "date": "[today]",
  "score": -1,
  "summary": "MU 27/27 analysts Strong Buy = crowded long. Analyst downgrades on ZS (Evercore to In-Line). Short interest rising in SPCX post-options listing.",
  "signals": [...],
  "crowded_positions": [
    {"ticker": "MU", "direction": "LONG", "crowding_level": "HIGH", "implication": "Reduce pre-earnings size 50%"}
  ]
}
```

---

## Rules

- Crowded positions must be flagged in counter_thesis.md Step D (Sentiment Risks)
- A 27/27 bullish analyst consensus is automatically flagged as HIGH crowding
- Options listing date is important: first options trading day = shorts can now enter → increased volatility
