# PROJECT STATE

- Current stage: E - Audit and Journal
- Last completed step: E3 - ReplayEngine journal wiring
- Next step: E4 - Trades and flat closes
- Code repo last commit: 934d7f5
- Docs repo last commit: (updated on push)

## Stage E progress
- E1 OK SQLite journal schema
- E2 OK Trace query
- E3 OK ReplayEngine journal wiring (per-bar transaction)
- E4    Trades and flat closes

## Test count
679 (A through E2) + 13 (E3) = 692 tests. CI green on every commit.

## Known issues
- submit_order remains public; execute_authorized is the production path.
- Regime classifier does not emit RISK_ON or RISK_OFF.
- AI provider is only the heuristic or a caller-supplied fixed provider.
- Trades table is not yet written by the engine. E3 writes market_events,
  fills, ai_decisions, validation_decisions, risk_decisions,
  portfolio_decisions, and orders. Trades and flat-close trades are E4.
- Journal has 16 tables; feature_snapshots, opportunities, market_regimes,
  news_events, positions, health_events, system_events are DDL-only. E4.
- Correlation-adjusted portfolio limits deferred to Stage F.
- Session verification deferred to historical qualification gate.
- CI dependencies unpinned. Stage F task.

## Open questions
- None.
