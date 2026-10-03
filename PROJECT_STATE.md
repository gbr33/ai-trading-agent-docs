# PROJECT STATE

- Current stage: F - Historical Qualification
- Last completed step: F1 Step 2 - IBKR downloader + first dataset
- Next step: F2 - One-day qualification replay
- Code repo last commit: (updated on push)
- Docs repo last commit: (updated on push)

## Stage F progress
- F1 Step 1 OK CSV loader/writer
- F1 Step 2 OK IBKR downloader + 390-bar dataset for 2026-09-08
- F2           One-day qualification replay (next)
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
- First date: 2026-09-08 (verified XNYS session, downloaded)
- Session: regular trading hours only
- Host: localhost (IPv6 loopback; macOS TWS binds IPv6)
- Port: 7497 (paper account API)
- Client ID: 1
- Read-only API, no startup fetch (StartupFetchNONE)

## The first dataset
- Path: data/raw/aapl-2026-09-08-1m.csv
- 390 bars, 09:30 to 15:59 ET
- AAPL prices in the 315-320 range
- Validated by app.data.quality.validate_dataset with zero issues
- Dataset id: aapl-2026-09-08-1m

## Test count
~760 tests. CI green on every commit.

## Known issues
- submit_order remains public; execute_authorized is the production path.
- The system has not yet run a single bar of real data through the
  ReplayEngine. F2 is the first time.
- IBKR data is unadjusted. Not an issue for a recent date.
- CI dependencies unpinned.

## Open questions
- None.
