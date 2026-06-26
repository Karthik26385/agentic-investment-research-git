# Signal Agent: Geopolitical / Trade

## Objective

Identify geopolitical and trade signals: tariffs, sanctions, commodity prices, peace deals, supply chain disruptions.

Runs in parallel with 4 other signal agents. Output writes to `state/signal_cache.json` under `geo_agent`.

---

## Searches to Run

1. "US China trade tariffs news today [month year]"
2. "Iran peace deal oil Strait of Hormuz update [month year]"
3. "geopolitical market risk today [date]"
4. "commodity prices oil gold copper today [date]"
5. "trade sanctions export controls news [month year]"

---

## Signals to Score

| Signal | BULL (+1) | NEUTRAL (0) | BEAR (−1) |
|--------|-----------|-------------|-----------|
| Oil price trend | Falling (consumer relief) | Stable | Rising (inflation risk) |
| Geopolitical risk | De-escalation / peace progress | Status quo | Escalation / new conflict |
| Trade tariff trend | Tariff reduction / deal progress | No change | Tariff escalation |
| Supply chain | Normalizing | Stable | Disruption / shortages |
| Commodity trend | Stable/falling | Mixed | Broad commodity spike |

---

## Ticker Exposure Mapping

For each major geopolitical event, identify directly exposed tickers:

```json
{
  "event": "Iran Hormuz reopening",
  "direction": "POSITIVE",
  "exposed_tickers": {
    "POSITIVE": ["DAL", "CCL", "UAL", "AMZN"],
    "NEGATIVE": ["XOM", "CVX", "FRO"]
  }
}
```

---

## Binary Event Tracking

Flag upcoming geopolitical binary dates:
- US-China tariff pause expiry
- Scheduled OPEC meetings
- Peace deal signing ceremonies
- Sanctions review dates

These dates feed into `state/signal_cache.json` → `earnings_agent.upcoming_binaries` (shared list).

---

## Output Format

```json
{
  "status": "complete",
  "date": "[today]",
  "score": 0,
  "summary": "Iran deal progress (WTI $79.20, Brent $81-84). US-China tariff pause expires ~Aug 10. Partial BULL from oil; BEAR from tariff risk.",
  "signals": [...],
  "exposed_tickers": [...],
  "upcoming_geo_binaries": [
    {"event": "US-China tariff pause expiry", "date": "2026-08-10", "risk": "HIGH", "affected": ["BABA", "BIDU", "TSM"]}
  ]
}
```

---

## Rules

- Oil price change > 5% is a HIGH-IMPORTANCE event — affects airlines, shipping, energy, consumer discretionary
- Any new military escalation = immediate BEAR signal regardless of other signals
- Trade deals that reduce tariffs = BULL signal for exposed sectors (tech supply chain, consumer goods)
