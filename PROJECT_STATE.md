# PROJECT STATE

- Current stage: D — Complete Replay
- Last completed step: D5 — News integration
- Next step: D6 — AI integration (strategy pipeline wiring)
- Code repo last commit: b78921a
- Docs repo last commit: (updated on push)

## Stage D progress
- D1 ✅ Historical event loop (app/backtest/engine.py)
- D2 ✅ Feature engine (app/features/technical.py)
- D3 ✅ Opportunity scanner (app/strategy/scanner.py)
- D4 ✅ Regime classifier (app/regime/engine.py)
- D5 ✅ News engine (app/news/engine.py)
- D6 ⏳ AI integration (strategy pipeline wiring)
- D7   Simulated execution
- D8   Position management
- D9   Exit management
- D10  Flattening
- D11  Reconciliation

## Test count
431 (A through D4) + 34 (D5) = 465 tests. CI green on every commit.

## Known issues
- submit_order remains public; execute_authorized is the production path.
- Regime classifier does not emit RISK_ON or RISK_OFF. Needs breadth or
  news data we do not have. Documented, not faked.
- News engine does not call external services. Records come in as a list.
  No AI-based sentiment. No prompt injection defense at this layer.
- Scanner, regime, news not yet wired into ReplayEngine.
- AI client not yet implemented. The AIProposal schema exists, but no
  provider produces one yet. D6.
- Exit management not yet implemented. D9.
- Correlation-adjusted portfolio limits deferred to Stage F.
- Session verification deferred to historical qualification gate.
- CI dependencies unpinned. Stage F task.

## Open questions
- None.
