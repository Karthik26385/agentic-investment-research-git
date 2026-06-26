# WebSearch (Built-in Tool)

**Status:** ✅ Active (allowed in `.claude/settings.local.json`)  
**Type:** Built-in Claude Code tool  
**Permission:** Pre-approved — no prompt required

---

## What It Does

Performs real-time web searches and returns structured results including titles, URLs, and summarized content. The primary data source for the entire pipeline when Robinhood MCP is not connected.

---

## Pipeline Usage

| Step | Purpose | Example Query |
|------|---------|---------------|
| Step 0.5 — Portfolio Review | Current price check for open positions | `"NVDA stock price June 23 2026"` |
| Step 1 — Macro Agent | Fed/CPI/PCE/yield data | `"Federal Reserve inflation CPI June 2026"` |
| Step 1 — Earnings Agent | Upcoming earnings, analyst consensus | `"Micron MU earnings preview June 24 2026 gross margin"` |
| Step 1 — Tech Agent | Index moves, VIX, breadth | `"Nasdaq S&P500 performance June 23 2026 semiconductors"` |
| Step 1 — Sentiment Agent | Analyst upgrades/downgrades | `"Wall Street analyst upgrades semiconductors June 2026"` |
| Step 1 — Geo Agent | Tariffs, geopolitical, oil | `"Iran tariffs trade war oil June 2026"` |
| Step 3 — Daily Brief | Company-specific news | `"SpaceX SPCX news June 23 2026"` |
| Step F — Price Validity | Verify assumed entry price | `"FDX stock price today June 23 2026"` |
| Step 7 — Context | Post-earnings reactions | `"FedEx earnings result after hours June 23 2026"` |

---

## Limitations

| Limitation | Impact | Mitigation |
|-----------|--------|-----------|
| Prices may be delayed 15–20 min | Step F accuracy reduced | Use Robinhood MCP when connected |
| No real-time order book | Cannot assess bid/ask spread | Accept slippage in paper trades |
| Rate limits on repeated queries | Cannot poll continuously | Run at session start only; Robinhood MCP for intraday |
| US-only results | Non-US market data may be limited | Korean/Asian market data via WebFetch to specific sites |
| No structured data return | Must parse natural language summaries | Use specific, targeted queries |

---

## Query Best Practices

```
# Good — specific, includes date
"NVDA Nvidia stock price June 23 2026"

# Good — includes ticker + context
"Micron MU Q3 2026 earnings gross margin analyst estimates"

# Avoid — too generic
"tech stocks today"
"what is happening in the market"
```

---

## Permission Config

Allowed via `.claude/settings.local.json`:
```json
{
  "permissions": {
    "allow": ["WebSearch"]
  }
}
```

No per-call prompt required.
