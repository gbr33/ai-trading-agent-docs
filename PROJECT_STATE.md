# PROJECT STATE

- Current stage: E - Audit and Journal
- Last completed step: E5 - Extended journal writes
- Next step: E6 - Experiment manifest persistence
- Code repo last commit: c4e7045
- Docs repo last commit: (updated on push)

## Stage E progress
- E1 OK SQLite journal schema
- E2 OK Trace query
- E3 OK ReplayEngine journal wiring
- E4 OK Trades and flat closes
- E5 OK Extended journal writes (features, regimes, news, opportunities)
- E6    Experiment manifest persistence

## Test count
707 (A through E4) + 7 (E5) = 714 tests. CI green on every commit.

## Known issues
- submit_order remains public; execute_authorized is the production path.
- Regime classifier does not emit RISK_ON or RISK_OFF.
- Journal tables positions, health_events, system_events remain DDL-only.
- experiments row is written on first bar when experiment_id is supplied
  but only records dataset_id, not a full manifest. E6.
- Correlation-adjusted portfolio limits deferred to Stage F.
- Session verification deferred to historical qualification gate.
- CI dependencies unpinned. Stage F task.

## Open questions
- None.
