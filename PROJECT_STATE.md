# PROJECT STATE

- Current stage: F - Historical Qualification
- Last completed step: F3 - One-week qualification (2026-09-08 to 2026-09-14)
- Next step: F4 - One-month qualification
- Code repo last commit: (updated on push)
- Docs repo last commit: (updated on push)

## Stage F progress
- F1 OK CSV loader, IBKR downloader, first dataset
- F2 OK One-day replay (2026-09-08)
- F3 OK One-week replay (2026-09-08 to 2026-09-14)
- F4    One-month qualification (next)
- F5    Multi-month / multi-year
- F6    Cost stress testing
- F7    Parameter sensitivity
- F8    Out-of-sample testing
- F9    Walk-forward testing
- F10   Benchmark comparison

## F3 results (AAPL, 5 trading days, 1-minute bars)

| Day | Bars | Opps | AI(BUY/HOLD) | Orders | Fills | Trades | P&L |
|-----|------|------|--------------|--------|-------|--------|-----|
| 09-08 | 390 | 5 | 1/4 | 0 | 0 | 0 | 0 |
| 09-09 | 390 | 4 | 3/1 | 1 | 2 | 1 | +163.44 |
| 09-10 | 390 | 5 | 5/0 | 1 | 2 | 1 | -51.82 |
| 09-11 | 390 | 1 | 1/0 | 0 | 0 | 0 | 0 |
| 09-14 | 390 | 5 | 4/1 | 1 | 2 | 1 | +98.43 |

Total: +210.05 over 3 closed trades on a $100k account.

Trade chain verified: each closed trade has decision_id and
market_event_id. trace_trade resolves all links.

## Test count
~830 tests. CI green on every commit.

## Known issues
- submit_order remains public; execute_authorized is the production path.
- Regime classifier does not emit RISK_ON or RISK_OFF.
- Structural coupling between heuristic confidence and validator
  minimum (F2 finding, unchanged).
- Exit reason (STOP_HIT / TARGET_HIT / etc.) is not stored in the
  trades table. Implicit in exit_price == plan.stop or plan.target.
- Exit prices carry full float precision. Live trading would quantize
  to tick size. F6 refinement.
- Zero friction: no spread, no slippage, no commission. P&L is an
  upper bound. F6 will stress it.
- IBKR data is unadjusted.
- CI dependencies unpinned.

## Open questions
- Does the +210.05 hold up under F6 cost stress? Unknown.
- Does the one-trade-per-day pattern persist at month scale? F4.
- Is the F2 structural coupling worth fixing before F5? Defer.
