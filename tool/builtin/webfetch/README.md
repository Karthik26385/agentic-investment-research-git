# WebFetch (Built-in Tool)

**Status:** ✅ Active  
**Type:** Built-in Claude Code tool  
**Permission:** Requires approval per call (not pre-approved)

---

## What It Does

Fetches the full content of a specific URL and returns it as text. Unlike WebSearch (which searches broadly), WebFetch retrieves a known page's structured content — useful for earnings filings, SEC documents, Fed statements, or data pages with stable URLs.

---

## Difference vs WebSearch

| | WebSearch | WebFetch |
|--|-----------|---------|
| Input | Query string | Specific URL |
| Best for | Finding information | Extracting from known source |
| Returns | Search result summaries | Full page content |
| Use case | "What happened to NVDA today?" | "Get the Fed's June 17 FOMC statement" |

---

## Pipeline Usage

| Step | Purpose | Example URL |
|------|---------|-------------|
| Step 1 — Macro Agent | Fed FOMC statements | `https://www.federalreserve.gov/monetarypolicy/fomcprojtabl20260617.htm` |
| Step 1 — Earnings Agent | SEC earnings filings | `https://www.sec.gov/cgi-bin/browse-edgar?...` |
| Step F — Price Validity | Yahoo Finance quote page | `https://finance.yahoo.com/quote/NVDA/` |
| Step 8 — Post-Market | Actual close prices from known aggregators | Investing.com historical data pages |

---

## Limitations

| Limitation | Impact |
|-----------|--------|
| JavaScript-rendered pages may not load | Some modern finance sites block scraping |
| Rate limits per domain | Avoid hammering the same site repeatedly |
| Login-gated content | Cannot access paywalled data (Bloomberg, WSJ) |
| URL must be known in advance | Requires prior knowledge of the target URL |

---

## Suggested Pre-Approved URLs for This Project

Add to `.claude/settings.local.json` to reduce approval prompts:

```json
{
  "permissions": {
    "allow": [
      "WebFetch(https://www.federalreserve.gov/*)",
      "WebFetch(https://finance.yahoo.com/quote/*)",
      "WebFetch(https://www.sec.gov/*)"
    ]
  }
}
```
