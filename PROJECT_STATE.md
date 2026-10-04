# PROJECT STATE

- Current stage: F - Historical Qualification
- Last completed step: F5 - Multi-month (Q3 2026, 64 sessions)
- Next step: F6 - Cost stress and dispersion analysis
- Code repo last commit: 65b15ee
- Docs repo last commit: (updated on push)

## Stage F progress
- F1 OK CSV loader, IBKR downloader
- F2 OK One-day (2026-09-08)
- F3 OK One-week (2026-09-08 to 2026-09-14)
- F4 OK One-month (September 2026, 21 sessions)
- F5 OK Multi-month (Q3 2026, 64 sessions)
- F6    Cost stress and dispersion analysis (next)
- F7    Parameter sensitivity
- F8    Out-of-sample testing
- F9    Walk-forward testing
- F10   Benchmark comparison

## F5 results (AAPL, 64 sessions, Q3 2026, zero friction)

- 23 trades total
- 12 wins / 11 losses (52.2% win rate)
- Total P&L: +390.65 on $100k

By month:
- July 2026: 22 sessions, 7 trades, +148.30
- August 2026: 21 sessions, 4 trades, +28.87
- September 2026: 21 sessions, 12 trades, +213.48

Determinism: September reproduces F4 exactly.

Concentration: three trades (07-17 +101.08, 09-09 +163.44, 09-14
+98.43) account for +362.95 of +390.65 total. Remove those three and
Q3 collapses to roughly +28.

## Test count
~830 tests. CI green on every commit.

## Known issues
- Strategy as configured produces +0.39% per quarter after zero
  friction. Under realistic costs the number will shrink materially.
- Return concentration is high. The edge may not be broad-based.
- Structural coupling between heuristic confidence and validator
  minimum (F2 finding) still live.
- Exit reason not stored in trades table.
- IBKR data unadjusted.
- CI dependencies unpinned.

## Open questions
- Does the +390.65 survive cost stress? F6.
- Is the edge broad or does it live in a handful of days? F6.
- Are the current thresholds near a robust region or a lucky point?
  F7.
- Does the strategy survive on unseen data? F8.
