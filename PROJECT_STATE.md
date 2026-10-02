# PROJECT STATE

- Current stage: D — Complete Replay
- Last completed step: D7 — Strategy pipeline wiring
- Next step: D8 — Position management
- Code repo last commit: 15a9c22
- Docs repo last commit: (updated on push)

## Stage D progress
- D1 OK Historical event loop
- D2 OK Feature engine
- D3 OK Opportunity scanner
- D4 OK Regime classifier
- D5 OK News engine
- D6 OK AI provider abstraction
- D7 OK Strategy pipeline (app/strategy/pipeline.py)
- D8    Position management
- D9    Exit management
- D10   Flattening
- D11   Reconciliation

## Test count
495 (A through D6) + 19 (D7) = 514 tests. CI green on every commit.

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
