# PROJECT STATE

- Current stage: D — Complete Replay
- Last completed step: D4 — Regime integration
- Next step: D5 — News integration
- Code repo last commit: e3c2e9e
- Docs repo last commit: (updated on push)

## Stage D progress
- D1 ✅ Historical event loop (app/backtest/engine.py)
- D2 ✅ Feature engine (app/features/technical.py)
- D3 ✅ Opportunity scanner (app/strategy/scanner.py)
- D4 ✅ Regime classifier (app/regime/engine.py)
- D5 ⏳ News integration
- D6   AI integration
- D7   Simulated execution
- D8   Position management
- D9   Exit management
- D10  Flattening
- D11  Reconciliation

## Test count
413 (A + B + C + D1 + D2 + D3) + 18 (D4) = 431 tests. CI green on every commit.

## Known issues
- submit_order remains public; execute_authorized is the production path.
- Regime classifier does not emit RISK_ON or RISK_OFF. Needs breadth or
  news data we do not have. Documented, not faked.
- Scanner, regime, news, AI not yet wired into ReplayEngine.
- Exit management not yet implemented. D9.
- Correlation-adjusted portfolio limits deferred to Stage F.
- Session verification deferred to historical qualification gate.
- CI dependencies unpinned. Stage F task.

## Open questions
- None.
