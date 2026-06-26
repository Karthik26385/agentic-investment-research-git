# System Prompt Command Reference
# AI-MarketResearch-Agent

**Last updated:** 2026-06-25
**Source:** `CLAUDE.md` (Operational Commands) + `workflows/orchestrator.md` (Trigger section)

> **Tip:** Append `; Granted all permission to bash and powershell` to any command to skip
> tool-use permission prompts and speed up pipeline runs.

---

## 1. Core Daily Triggers

Full 9-step pipeline. Mode must already be set, or use the explicit mode variants below.

```
start
start SIMULATION
start RESEARCH
start SIMULATION with full deployable $100
SIMULATION - Run
```

---

## 2. Backtesting Variants

Switches to BACKTESTING mode and runs the historical simulation engine for N trading days.
Default universe: 24 tickers across 8 S&P 500 sectors. Results are plumbing/logic tests only
(v1.0 has unavoidable LLM look-ahead bias — see readMe/Solution_Document.md Section 11).

```
Start BACKTESTING 5 Days
Start BACKTESTING 10 Days
Start BACKTESTING 20 Days
Start BACKTESTING 30 Days
Start BACKTESTING
```

Custom universe example:
```
Start BACKTESTING 10 Days with universe: MRVL, CRWD, ISRG, EOG, ABNB
```

---

## 3. Mode Switching

Switches the active mode in `state/mode.json` and runs the full pipeline.

```
switch to RESEARCH
switch to SIMULATION
switch to EXECUTION
```

With permission grant (skips all tool prompts — recommended for pipeline runs):
```
start RESEARCH; Granted all permission to bash and powershell
start SIMULATION; Granted all permission to bash and powershell
Start BACKTESTING 10 Days; Granted all permission to bash and powershell
```

> **Note:** `switch to EXECUTION` validates safety gates first:
> requires >=10 simulation days + win rate >=45% + Robinhood MCP connected.
> BACKTESTING days do NOT count toward this gate.

---

## 4. Portfolio Management

Commands targeting Step 0.5 (portfolio review) and portfolio state.

```
Run portfolio review
Check my open positions
Run post market review
Run intraday monitoring
Show me the equity curve
What is my current P&L?
```

> **When to use:**
> - `Run portfolio review` — mid-session stop check, anytime during market hours
> - `Run intraday monitoring` — if a position has moved +-3% since last check
> - `Run post market review` — after 4 PM ET to lock in official close prices and update all state files

---

## 5. Step-Specific (Partial Pipeline)

Runs a single pipeline step without executing the full orchestrator.

```
Run signal agents only
What is the regime today?
Run counter thesis on [TICKER] LONG
Run counter thesis on [TICKER] SHORT
Show me the top 5 candidates
Check sector exposure
```

Replace `[TICKER]` with the actual symbol (e.g., `Run counter thesis on NVDA LONG`).

---

## 6. Special Operations

```
Start SIMULATION with full deployable $100; Granted all permission to bash and powershell
switch to EXECUTION
Run post market review and update all state files
Send daily email report
Prepare GitHub release v[VERSION]
```

---

## 7. Quick Reference (8 Pinned Commands)

| # | Command | When to Use |
|---|---------|-------------|
| 1 | `start SIMULATION` | Every morning — daily paper trading run |
| 2 | `start RESEARCH` | Analysis without committing paper capital |
| 3 | `Start BACKTESTING 10 Days` | After any strategy change — stress-test the gates |
| 4 | `start SIMULATION with full deployable $100` | Fresh portfolio reset to $100 |
| 5 | `Run portfolio review` | Mid-session stop check on open positions |
| 6 | `Run post market review` | After 4 PM ET — lock in close prices, update state |
| 7 | `Run intraday monitoring` | Position moved +-3% during market hours |
| 8 | `switch to EXECUTION` | Going live (requires >=10 sim days + >=45% win rate) |

---

## Mode -> Output Directory Mapping

| Mode | Trigger | Output Location |
|------|---------|----------------|
| SIMULATION | `start SIMULATION` | `FinalResearchoutput/simulation/YYYY-MM-DD/` |
| RESEARCH | `start RESEARCH` | `FinalResearchoutput/research/YYYY-MM-DD/` |
| BACKTESTING | `Start BACKTESTING N Days` | `FinalResearchoutput/backtesting/YYYY-MM-DD/` |
| EXECUTION | `switch to EXECUTION` | `FinalResearchoutput/live/YYYY-MM-DD/` |

All output files use the format: `[DATE]_[SID]_[FileName].md`
where SID = session start time in HHMMSS (e.g., `2026-06-25_130001_Master_Summary.md`)

---

## GitHub Publish

To push a new version to the public GitHub repository:

```
Prepare GitHub release v[VERSION]
```

Claude will copy + sanitize all files from the live system into the
`agentic-investment-research-git/` sibling directory and report a diff summary.
Then run manually:

```bash
cd ../agentic-investment-research-git
git add .
git commit -m "Release vX.X — [description]"
git push origin main
```
