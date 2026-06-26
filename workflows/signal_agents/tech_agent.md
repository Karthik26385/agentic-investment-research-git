# Signal Agent: Technical / Price

## Objective

Identify market structure signals: index trends, breadth, volume, VIX, sector rotation.

Runs in parallel with 4 other signal agents. Preferentially uses Robinhood MCP for live prices when connected; falls back to WebSearch.

Output writes to `state/signal_cache.json` under `tech_agent`.

---

## Data to Collect

### If Robinhood MCP connected:
- `get_quote("SPY")` → S&P 500 ETF price/change
- `get_quote("QQQ")` → Nasdaq-100 price/change
- `get_quote("IWM")` → Russell 2000 price/change
- `get_quote("VIX")` → Volatility index
- `get_quote("XLK")` → Tech sector ETF
- `get_quote("XLF")` → Financials (rate-sensitive)
- `get_quote("XLE")` → Energy

### If WebSearch only:
- Search: "S&P 500 Nasdaq Russell 2000 performance today [date]"
- Search: "VIX volatility index today [date]"
- Search: "sector ETF performance today XLK XLF XLE [date]"

---

## Signals to Score

| Signal | BULL (+1) | NEUTRAL (0) | BEAR (−1) |
|--------|-----------|-------------|-----------|
| S&P 500 trend | Up > 0.5% | Flat ±0.5% | Down > 0.5% |
| Nasdaq vs Dow spread | Nasdaq leads | Equal | Dow leads / Nasdaq lags |
| Russell 2000 | Rising (breadth expanding) | Flat | Falling |
| VIX | Below 15 and falling | 15-20 stable | Above 20 or rising |
| Sector rotation | Tech/growth leading | Mixed | Tech selling; defensives leading |
| Breakouts working | New highs across sectors | Some sectors | Breakouts failing / reversals |

---

## Breadth Detection

If Nasdaq and Dow diverge by > 1%:
- Dow > Nasdaq: rotation signal → tag "ROTATION" in summary
- Nasdaq > Dow: risk-on signal → BULL modifier

If Russell 2000 outperforms S&P by > 1.5%: small-cap rotation signal → potential late BULL cycle

---

## Output Format

```json
{
  "status": "complete",
  "date": "[today]",
  "score": -2,
  "summary": "S&P -0.37%, Nasdaq -1.32%, Dow +0.29%. VIX 16.78. Tech selling; industrial rotation. CRWV fell despite index inclusion.",
  "signals": [...],
  "data_source": "WebSearch",
  "breadth_note": "Dow-Nasdaq divergence: +1.61% → rotation from tech to industrials"
}
```

---

## Rules

- Always note the data source (Robinhood MCP vs WebSearch)
- If Robinhood MCP is connected, always prefer it for price data (exact vs approximate)
- Sector divergence > 2% in either direction is a HIGH-IMPORTANCE signal — note in summary
