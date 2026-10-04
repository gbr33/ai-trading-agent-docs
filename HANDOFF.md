# HANDOFF — read this first in any new chat

## How to orient in a new chat

Before doing anything, read in this order:
1. This file.
2. PROJECT_STATE.md
3. NEXT_STEP.md
4. DECISIONS.md (last ~500 lines)
5. docs/hypotheses/orb-aapl-1m.md and orb-aapl-1m-v2.md in the code
   repo — these are the pre-registration discipline examples.

Then confirm to the user: "Handoff read. Current stage: F complete.
Option 2 chosen: new hypothesis. Ready to draft."

## Two repos

- Public docs repo (state only, no secrets):
  https://github.com/gbr33/ai-trading-agent-docs
- Private code repo (all source, all tests):
  https://github.com/gbr33/ai-trading-agent

## Where the project is

Stages A through E are complete. That is the entire AI trading
system platform: data, features, regime, scanner, news, AI interface,
validator, risk, portfolio, broker abstraction, fill engine, exit
engine, flatten engine, reconciliation, journal, deterministic replay.

Stage F (historical qualification) is complete. Two strategy families
were tested against real market data with a realistic friction model.
Both failed.

No surviving strategy. The platform is the asset.

## The platform (all of Stages A-E)

- app/data: MarketEvent, HistoricalDataProvider, PointInTimeEventStream,
  CSV loader/writer, dataset validator
- app/runtime: SimulationClock with XNYS session derivation
- app/features: RSI, RVOL, ATR, momentum, SMA, volatility, trend
- app/strategy: opportunity scanner, ORB strategy, heuristic pipeline
- app/regime: deterministic regime classifier
- app/news: point-in-time news engine
- app/ai: AIProposal schema, AIProvider ABC, HeuristicProvider,
  fail-safe wrapper
- app/validation: deterministic decision validator
- app/risk: deterministic risk engine with sizing and exposure caps
- app/portfolio: deterministic portfolio engine
- app/execution: Broker ABC, SimulatedBroker, ExecutionAuthorization,
  OrderManager, ExitEngine, FlatEngine, ReconciliationEngine,
  order state machine
- app/backtest: SimulatedAccount, FillModel, try_fill, ReplayEngine
- app/journal: SQLite journal, 16 tables, 13 typed insert/get pairs,
  trace_trade, ExperimentManifest

~880 tests. CI green on every commit. Deterministic replay verified
across 126 sessions (Q2 + Q3 2026).

## Stage F results (both falsified)

Heuristic momentum screen, Q3 2026 (64 sessions):
- zero: +390.65
- normal friction: -22.08 (falsified, Blueprint Section 42)

ORB v1, Q3 2026 (in-sample, 64 sessions):
- zero: +3425.44
- normal friction: +2851.30
- failed pre-registered R/R criterion (1.49 vs 1.50 bar)

ORB v1, Q2 2026 (exploratory out-of-sample, 62 sessions):
- zero: +323.45
- normal friction: -303.79 (falsified)

ORB v2 (no target), Q2 2026 (62 sessions):
- zero: +4.05
- normal friction: -623.09 (falsified in-sample)

## Working discipline (critical)

The user and instructor work in strict steps. Each step is:

1. Instructor issues a step contract:
   - Goal (one sentence)
   - Files to create/modify (canonical paths only)
   - Tests that prove it
   - Acceptance criteria (binary)
   - Rollback plan
   - Design decisions (numbered)

2. User replies "approved" or requests a change.

3. Instructor gives ONE command. The user pastes it whole.

4. User pastes the full output.

5. If red: instructor gives ONE fix command. Repeat.

6. If green: instructor gives ONE commit command, then user
   confirms CI on GitHub.

7. Instructor updates the docs repo.

8. Next step.

Rules:
- One step per reply. Never multiple steps or parallel tracks.
- Never create a file not in Blueprint Section 5 without asking
  and getting explicit approval.
- Never use a synonym path. Canonical paths only.
- No code without approval.
- Any strategy work uses a pre-registered hypothesis document
  before any backtest. The hypothesis says success criteria and
  falsification criteria in advance. We run in-sample once. If
  it passes, we run out-of-sample on fresh data. We never tune
  after seeing the result.
- Checks always in this order: ruff check --fix, ruff check,
  pytest, mypy.
- Code is deterministic. No datetime.now() in trading logic.
- Fail closed. Uncertain means no trade.

## TWS / IBKR connection (macOS-specific, do not lose this)

The user has IBKR Trader Workstation running locally on macOS
12.7.6 Intel. Paper account API on port 7497.

Key facts discovered the hard way:

1. **macOS TWS binds IPv6 only.** lsof shows `IPv6 *:7497 (LISTEN)`.
   It does NOT accept IPv4 connections on 127.0.0.1.

2. **Use `--host localhost`, not `--host 127.0.0.1`.** `localhost`
   resolves to `::1` on macOS. That is what TWS accepts.
   Connecting to 127.0.0.1 times out with errno 60.

3. **TWS API must be enabled:**
   File > Global Configuration > API > Settings
   - Check "Enable ActiveX and Socket Clients"
   - Socket port: 7497 (paper) / 7496 (live)
   - Trusted IPs: add 127.0.0.1 AND localhost as separate lines
   - Apply, OK, restart TWS if prompted

4. **Python library: `ib_async` 2.1.0** (the maintained fork of
   ib_insync, whose author died in 2024).

5. **Connect with `readonly=True` and `fetchFields=StartupFetchNONE`.**
   Without StartupFetchNONE, the post-connection sync phase hangs
   on a fresh TWS session.

6. **Timeout 30–60 seconds.** 10 is too short on first connect.

7. **Script: `scripts/download_ibkr_bars.py`**

   Single day:
   python scripts/download_ibkr_bars.py --symbol AAPL --date 2026-09-08

   Range (skips weekends and holidays via exchange_calendars XNYS):
   python scripts/download_ibkr_bars.py --symbol AAPL \
       --start 2026-04-01 --end 2026-06-30

   Output: data/raw/<symbol>-<start>_to_<end>-1m.csv

8. **IBKR pacing:** roughly 60 historical requests per 10 minutes.
   A 62-session range takes ~11 minutes. Disconnects mid-range
   are possible; re-run the remaining range if it happens.

9. **Data is unadjusted.** For recent dates with no splits or
   dividends, this is fine. For older data, it matters.

10. **1-minute TRADES bars, useRTH=True.** Regular hours only.
    IBKR returns 390 bars per session.

## Key scripts (all in scripts/)

- download_ibkr_bars.py — IBKR downloader (single day or range)
- split_week.py — split a multi-day CSV into one CSV per session
- replay_one_day.py — replay one session through the engine
- cost_stress.py — run a whole dataset at 4 friction levels
- compare_journals.py — aggregate journals into a table
- trade_dispersion.py — distribution of trade P&L

## Friction model (in FillModel, app/backtest/fills.py)

- zero: 0 bps spread, 0 bps slippage, $0 commission
- normal: 2 bps, 2 bps, $0.005/share, $1 min
- moderate: 5 bps, 5 bps, $0.005/share, $1 min
- high: 10 bps, 10 bps, $0.01/share, $1 min

Applied to BOTH entries and exits (asymmetric: adverse for each side).

## Next step: Option 2 chosen

New hypothesis. The instructor proposed overnight gap fade as the
most honest next hypothesis (different mechanism, one trade per
day, small friction fraction). Not yet drafted.

The next chat's job: draft the pre-registration document for a new
hypothesis (probably overnight gap fade, but the user decides),
get approval, then implement.

Do not proceed without a written hypothesis document. Do not tune
any existing parameters. Do not modify ORB rules.
