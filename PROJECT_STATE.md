# PROJECT STATE

- Current stage: D — Complete Replay
- Last completed step: D1 — Historical event loop
- Next step: D2 — Strategy pipeline integration (scanner, regime, news, AI)
- Code repo last commit: 7c10273
- Docs repo last commit: (updated on push)

## Deliverables so far
Stage A:
- app/data/models.py, app/data/historical.py, app/data/quality.py, app/runtime/clock.py

Stage B:
- app/backtest/accounting.py, app/backtest/fills.py
- app/execution/state_machine.py, app/execution/broker.py, app/execution/simulated.py

Stage C (complete):
- app/ai/engine.py, app/validation/decision.py, app/risk/engine.py
- app/portfolio/controller.py
- app/execution/broker.py (ExecutionAuthorization), app/execution/simulated.py (execute_authorized)

Stage D:
- app/backtest/engine.py — event-driven replay loop

## Test count
333 (A + B + C) + 15 (D1) = 348 tests. CI green on every commit.

## Known issues
- submit_order remains public. execute_authorized is the production path.
  Closing the gap on submit_order is deferred to a later D stage when the
  orchestrator takes over and submit_order becomes private.
- Scanner, regime, news, AI integration not yet wired into the engine.
  The engine accepts a strategy callback. D2 fills it in.
- Exit management (stops, targets, flatten) not yet implemented. D stage
  later step.
- Correlation-adjusted portfolio limits deferred to Stage F.
- Session verification deferred to historical qualification gate.
- CI dependencies unpinned. Stage F task.

## Open questions
- None.
