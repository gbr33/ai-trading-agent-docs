# PROJECT STATE

- Current stage: D — Complete Replay
- Last completed step: C4 — Execution authorization (Stage C complete)
- Next step: D1 — Historical event loop
- Code repo last commit: 3ffe76b
- Docs repo last commit: (updated on push)

## Deliverables so far
Stage A:
- app/data/models.py, app/data/historical.py, app/data/quality.py, app/runtime/clock.py

Stage B:
- app/backtest/accounting.py, app/backtest/fills.py
- app/execution/state_machine.py, app/execution/broker.py, app/execution/simulated.py

Stage C (complete):
- app/ai/engine.py            — AI proposal schema
- app/validation/decision.py  — deterministic decision validator
- app/risk/engine.py          — deterministic risk engine with sizing
- app/portfolio/controller.py — deterministic portfolio engine
- app/execution/broker.py     — ExecutionAuthorization + build_authorization
- app/execution/simulated.py  — execute_authorized (production path)

## Test count
309 (A + B + C1 + C2 + C3) + 24 (C4) = 333 tests. CI green on every commit.

## Known issues
- submit_order is still public and does not require authorization. The
  production path execute_authorized does require it. Closing this gap is a
  Stage D Step 1 task when the orchestrator takes over.
- Correlation-adjusted portfolio limits deferred to Stage F.
- Session verification deferred to historical qualification gate.
- CI dependencies unpinned. Stage F task.

## Open questions
- None.
