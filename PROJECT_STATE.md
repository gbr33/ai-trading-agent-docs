# PROJECT STATE

- Current stage: F - Historical Qualification
- Last completed step: F4 - One-month qualification (September 2026)
- Next step: F5 - Multi-month / multi-year
- Code repo last commit: (updated on push)
- Docs repo last commit: (updated on push)

## Stage F progress
- F1 OK CSV loader, IBKR downloader, first dataset
- F2 OK One-day replay (2026-09-08)
- F3 OK One-week replay (2026-09-08 to 2026-09-14)
- F4 OK One-month replay (September 2026, 21 sessions)
- F5    Multi-month / multi-year (next)
- F6    Cost stress testing
- F7    Parameter sensitivity
- F8    Out-of-sample testing
- F9    Walk-forward testing
- F10   Benchmark comparison

## F4 results (AAPL, 21 trading days, September 2026)

- 21 days scanned
- 12 trades total
- 6 wins / 6 losses (50.0% win rate)
- Total P&L: +213.48 on $100k
- 10 of 21 days had zero trades
- 09-30 had zero opportunities

Determinism: the 5 F3 days reproduced exactly.

F3 alone was +210.05 over 3 trade-days.
F4 total is +213.48 over 11 trade-days.
So the 16 days not in F3 contributed only +3.43.

Interpretation: F3 was noise. The full month is flat after
zero friction. Under realistic friction the result is likely
negative.

## Test count
~830 tests. CI green on every commit.

## Known issues
- Strategy as currently configured does not show meaningful edge on
  AAPL 1-minute bars in September 2026.
- Structural coupling between heuristic confidence and validator
  minimum (F2 finding) drives many rejections.
- One-trade-per-day pattern is implicit, not by design. Worth
  understanding before F5.
- Zero friction. F6 will stress it.
- Exit reason not stored in trades table.
- IBKR data unadjusted.

## Open questions
- Is September 2026 representative? F5 will answer.
- Does the one-trade-per-day cap come from risk, portfolio, or
  validation? Worth diagnosing.
- Should the F2 structural coupling be fixed before F5? Lean yes,
  but not urgent.
