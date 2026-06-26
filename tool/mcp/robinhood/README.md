# Robinhood MCP

**Status:** 🟡 Not yet registered — run the setup command below and restart Claude Code
**Server name:** `robinhood-trading`
**Endpoint:** `https://agent.robinhood.com/mcp/trading`
**Transport:** HTTP
**Priority:** HIGH — required for EXECUTION mode; improves SIMULATION accuracy

---

## Why This Matters

Without Robinhood MCP, the pipeline uses WebSearch for price data — approximate and delayed. All price checks (Step F), stop monitoring, and post-market close prices rely on this tool when connected. EXECUTION mode (real orders) **requires** it.

| Mode | Without Robinhood | With Robinhood |
|------|-------------------|----------------|
| RESEARCH | WebSearch fallback (OK) | Live quotes |
| SIMULATION | WebSearch fallback (OK) | Live quotes, accurate stops |
| EXECUTION | ❌ Cannot place orders | ✅ Live quotes + order execution |

---

## Setup

```bash
# Register the MCP server (run once per machine / Claude Code installation)
claude mcp add robinhood-trading --transport http https://agent.robinhood.com/mcp/trading

# Restart Claude Code to activate

# Verify
claude mcp list
# Expected output includes:
# robinhood-trading: https://agent.robinhood.com/mcp/trading - ✔ Connected

# Remove if needed
claude mcp remove robinhood-trading
```

The server is registered at project scope in your local `~/.claude.json`. It is only active within this project directory.

---

## Pipeline Integration

| Step | Usage |
|------|-------|
| Step F (Price Validity) | `get_quote(ticker)` → verify assumed entry price is within 10% of live price |
| Step 0.5 (Portfolio Review) | `get_positions()` → sync paper positions with live prices |
| Step 7 (Intraday Monitoring) | `get_quote(ticker)` → check stops during market hours |
| Step 8 (Post-Market Review) | `get_historical_prices(ticker, interval='day')` → actual close price |
| EXECUTION trades | `place_order(ticker, side, qty, type)` → after human confirms |

---

## Expected Tools

| Tool | Description | Mode |
|------|-------------|------|
| `get_quote` | Live bid/ask/last price for a ticker | All modes |
| `get_portfolio` | Account equity, buying power, cash | EXECUTION |
| `get_positions` | All open positions with current prices | EXECUTION |
| `place_order` | Submit market/limit/stop order | EXECUTION only |
| `cancel_order` | Cancel a pending order | EXECUTION |
| `get_order_status` | Check order fill status | EXECUTION |
| `get_historical_prices` | OHLCV data (day/5min intervals) | All modes |
| `get_options_chain` | Options quotes (not currently used) | Future |

---

## Safety Gates (EXECUTION Mode)

All order placement goes through mandatory gates before any `place_order` call:

1. `active_mode == EXECUTION` — orders blocked in RESEARCH and SIMULATION
2. `counter_thesis_rating != STRONG` — STRONG CT always blocks entry
3. `regime != BEAR` if trade is LONG — BEAR regime blocks long entries
4. `portfolio.max_drawdown_pct < 5%` — halt all new trades if drawdown ≥ 5%
5. **Human confirmation** — display full trade card, await explicit `CONFIRM` response

---

## Known Limitations

- Robinhood does not support short selling for most retail accounts — Step 8b converts SHORT theses to inverse ETFs or put options
- Options are available but not currently part of the pipeline (require explicit enablement on your Robinhood account)
- API rate limits: ~100 requests/minute
- Token refresh required approximately every 24 hours
