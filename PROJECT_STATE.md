# PROJECT STATE

- Current stage: F - Historical Qualification
- Last completed step: F1 Step 1 - CSV loader/writer
- Next step: F1 Step 2 - IBKR historical downloader
- Code repo last commit: 0a3ff67
- Docs repo last commit: (updated on push)

## Stage F progress
- F1 Step 1 OK CSV loader/writer (app/data/historical.py)
- F1 Step 2    IBKR historical downloader (script)
- F2           One-day qualification
- F3           One-week qualification
- F4           One-month qualification
- F5           Multi-month / multi-year
- F6           Cost stress testing
- F7           Parameter sensitivity
- F8           Out-of-sample testing
- F9           Walk-forward testing
- F10          Benchmark comparison

## Locked decisions for Stage F
- Vendor: IBKR via TWS API (F1-F4; re-evaluate at F5)
- Universe: AAPL only
- Timeframe: 1-minute bars
- First date: 2026-09-08 (verified XNYS session)
- Session: regular trading hours only (09:30-16:00 ET)

## Test count
~760 tests. CI green on every commit.

## Known issues
- submit_order remains public; execute_authorized is the production path.
- Regime classifier does not emit RISK_ON or RISK_OFF.
- AI provider is only heuristic or a caller-supplied fixed provider.
- The system has never seen real market data. F2 is the first real test.
- IBKR integration does not exist yet. F1 Step 2.
- CI dependencies unpinned.

## Open questions
- None.
