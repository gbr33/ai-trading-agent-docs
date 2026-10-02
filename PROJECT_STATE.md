# PROJECT STATE

- Current stage: D — Complete Replay
- Last completed step: D8 — Position manager
- Next step: D9 — Exit management
- Code repo last commit: b264cb3
- Docs repo last commit: (updated on push)

## Stage D progress
- D1 OK Historical event loop
- D2 OK Feature engine
- D3 OK Opportunity scanner
- D4 OK Regime classifier
- D5 OK News engine
- D6 OK AI provider abstraction
- D7 OK Strategy pipeline
- D8 OK Position manager (app/execution/order_manager.py)
- D9    Exit management
- D10   Flattening
- D11   Reconciliation

## Test count
514 (A through D7) + 29 (D8) = 543 tests. CI green on every commit.

## Known issues
- submit_order remains public; execute_authorized is the production path.
- Regime classifier does not emit RISK_ON or RISK_OFF.
- News engine does not call external services. No AI-based sentiment.
- AI provider is only the heuristic. No real LLM provider yet.
- Exit management not yet implemented. D9.
- Correlation-adjusted portfolio limits deferred to Stage F.
- Session verification deferred to historical qualification gate.
- CI dependencies unpinned. Stage F task.

## Open questions
- None.
