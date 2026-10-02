# PROJECT STATE

- Current stage: E - Audit and Journal
- Last completed step: E2 - Trace query
- Next step: E3 - ReplayEngine journal wiring
- Code repo last commit: afbe1b4
- Docs repo last commit: (updated on push)

## Stage E progress
- E1 OK SQLite journal schema
- E2 OK Trace query (trade -> market event chain)
- E3    ReplayEngine journal wiring
- E4    Experiment manifest persistence

## Test count
666 (A through E1) + 13 (E2) = 679 tests. CI green on every commit.

## Known issues
- submit_order remains public; execute_authorized is the production path.
- Regime classifier does not emit RISK_ON or RISK_OFF.
- AI provider is only the heuristic. No real LLM provider yet.
- ReplayEngine does not write to the journal yet. E3.
- Journal has 16 tables; 9 have typed insert/get pairs. The other 7 have
  DDL only.
- Correlation-adjusted portfolio limits deferred to Stage F.
- Session verification deferred to historical qualification gate.
- CI dependencies unpinned. Stage F task.

## Open questions
- None.
