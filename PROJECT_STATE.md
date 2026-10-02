# PROJECT STATE

- Current stage: E - Audit and Journal
- Last completed step: E4 - Trades and flat closes
- Next step: E5 - Extended journal writes (features, regimes, news, opportunities, positions)
- Code repo last commit: 6f81abf
- Docs repo last commit: (updated on push)

## Stage E progress
- E1 OK SQLite journal schema
- E2 OK Trace query
- E3 OK ReplayEngine journal wiring
- E4 OK Trades and flat closes
- E5    Extended journal writes
- E6    Experiment manifest persistence

## Test count
692 (A through E3) + 15 (E4) = 707 tests. CI green on every commit.

## Known issues
- submit_order remains public; execute_authorized is the production path.
- Regime classifier does not emit RISK_ON or RISK_OFF.
- AI provider is only the heuristic or a caller-supplied fixed provider.
- Journal tables feature_snapshots, opportunities, market_regimes,
  news_events, positions, health_events, system_events are DDL-only.
  E5.
- experiments table is written on first bar with experiment_id, but no
  manifest config is captured. E6.
- Correlation-adjusted portfolio limits deferred to Stage F.
- Session verification deferred to historical qualification gate.
- CI dependencies unpinned. Stage F task.

## Open questions
- None.
