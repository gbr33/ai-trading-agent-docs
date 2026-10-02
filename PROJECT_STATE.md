# PROJECT STATE

- Current stage: D — Complete Replay
- Last completed step: D6 — AI provider abstraction and heuristic provider
- Next step: D7 — Strategy module wiring into ReplayEngine
- Code repo last commit: 2e16705
- Docs repo last commit: (updated on push)

## Stage D progress
- D1 OK Historical event loop (app/backtest/engine.py)
- D2 OK Feature engine (app/features/technical.py)
- D3 OK Opportunity scanner (app/strategy/scanner.py)
- D4 OK Regime classifier (app/regime/engine.py)
- D5 OK News engine (app/news/engine.py)
- D6 OK AI provider abstraction (app/ai/engine.py)
- D7    Strategy module wiring into ReplayEngine
- D8    Position management
- D9    Exit management
- D10   Flattening
- D11   Reconciliation

## Test count
465 (A through D5) + 30 (D6) = 495 tests. CI green on every commit.

## Known issues
- submit_order remains public; execute_authorized is the production path.
- Regime classifier does not emit RISK_ON or RISK_OFF.
- News engine does not call external services. No AI-based sentiment.
- AI provider is only the heuristic. No real LLM provider yet.
- Scanner, regime, news, AI not yet wired into ReplayEngine. D7.
- Exit management not yet implemented. D9.
- Correlation-adjusted portfolio limits deferred to Stage F.
- Session verification deferred to historical qualification gate.
- CI dependencies unpinned. Stage F task.

## Open questions
- None.
