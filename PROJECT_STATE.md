# PROJECT STATE

- Current stage: D - Complete Replay
- Last completed step: D10 - Flatten engine
- Next step: D11 - Reconciliation
- Code repo last commit: fda5718
- Docs repo last commit: (updated on push)

## Stage D progress
- D1  OK Historical event loop
- D2  OK Feature engine
- D3  OK Opportunity scanner
- D4  OK Regime classifier
- D5  OK News engine
- D6  OK AI provider abstraction
- D7  OK Strategy pipeline
- D8  OK Position manager
- D9  OK Exit engine
- D10 OK Flatten engine (app/execution/flatten.py)
- D11    Reconciliation

## Test count
577 (A through D9) + 49 (D10) = 626 tests. CI green on every commit.

## Known issues
- submit_order remains public; execute_authorized is the production path.
- Regime classifier does not emit RISK_ON or RISK_OFF.
- AI provider is only the heuristic. No real LLM provider yet.
- Correlation-adjusted portfolio limits deferred to Stage F.
- Session verification deferred to historical qualification gate.
- CI dependencies unpinned. Stage F task.

## Open questions
- None.
