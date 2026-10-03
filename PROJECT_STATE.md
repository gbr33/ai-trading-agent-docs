# PROJECT STATE

- Current stage: F - Historical Qualification
- Last completed step: F2 - One-day qualification replay (2026-09-08)
- Next step: F3 - One-week qualification
- Code repo last commit: (updated on push)
- Docs repo last commit: (updated on push)

## Stage F progress
- F1 OK CSV loader, IBKR downloader, first dataset
- F2 OK One-day qualification replay (2026-09-08)
- F3    One-week qualification (next)
- F4    One-month qualification
- F5    Multi-month / multi-year
- F6    Cost stress testing
- F7    Parameter sensitivity
- F8    Out-of-sample testing
- F9    Walk-forward testing
- F10   Benchmark comparison

## F2 results on 2026-09-08 (AAPL)
- 390 bars processed
- 331 feature snapshots, 331 regimes
- 5 opportunities (RVOL 3.04 to 7.95)
- 5 AI decisions: 4 HOLD, 1 BUY
- 1 validation, rejected for confidence 0.6287 < 0.70
- 0 orders, 0 fills, 0 trades
- Journal complete: every layer recorded

## F2 findings
1. Pipeline works end to end on real data. No crashes, no silent
   failures, journal complete.
2. Scanner is well-tuned. 5 opportunities per day, RVOL > 3 filter
   doing real work on a normal AAPL day.
3. Heuristic's required_trend=UP gate is the dominant filter. 3 of 4
   HOLDs fired because SMA9 < SMA21, even though RSI was 55-69.
4. Structural coupling: heuristic confidence == opportunity score, so
   BUYs with score in [0.50, 0.70) are always rejected by the
   validator. This wastes work and hides intent. Recorded for F5+.

## Test count
~800 tests. CI green on every commit.

## Known issues
- submit_order remains public; execute_authorized is the production path.
- Regime classifier does not emit RISK_ON or RISK_OFF.
- Structural coupling between heuristic confidence and validator
  minimum (see F2 findings).
- The one BUY on 2026-09-08 was rejected by the validator. No trade
  was placed. This is the correct fail-closed behavior.
- IBKR data is unadjusted. Not an issue for a recent date.
- CI dependencies unpinned.

## Open questions
- The structural coupling needs a decision before Stage F ends.
  Options A (accept), B (heuristic self-throttle), C (decouple
  confidence from score). Decided with F3 data, not F2's n=1.
