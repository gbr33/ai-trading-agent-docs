# PROJECT STATE

- Current stage: E - Audit and Journal
- Last completed step: E1 - SQLite journal schema
- Next step: E2 - Trace query (Blueprint Section 32)
- Code repo last commit: 92dd8a2
- Docs repo last commit: (updated on push)

## Stage E progress
- E1 OK SQLite journal schema (app/journal/database.py)
- E2    Trace query (full trade -> market event chain)
- E3    ReplayEngine journal wiring
- E4    Experiment manifest persistence

## Test count
640 (A through D11) + 26 (E1) = 666 tests. CI green on every commit.

## Known issues
- submit_order remains public; execute_authorized is the production path.
- Regime classifier does not emit RISK_ON or RISK_OFF.
- AI provider is only the heuristic. No real LLM provider yet.
- Journal has 16 tables; 9 have typed insert/get pairs (audit spine).
  The other 7 have DDL only. Typed methods are E2.
- ReplayEngine does not write to the journal yet. E3.
- Correlation-adjusted portfolio limits deferred to Stage F.
- Session verification deferred to historical qualification gate.
- CI dependencies unpinned. Stage F task.

## Open questions
- None.
