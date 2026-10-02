# PROJECT STATE

- Current stage: C — Common Decision Pipeline
- Last completed step: C3 — Portfolio engine
- Next step: C4 — Execution authorization
- Code repo last commit: 7400aef
- Docs repo last commit: (updated on push)

## Deliverables so far
Stage A:
- app/data/models.py, app/data/historical.py, app/data/quality.py, app/runtime/clock.py

Stage B:
- app/backtest/accounting.py, app/backtest/fills.py
- app/execution/state_machine.py, app/execution/broker.py, app/execution/simulated.py

Stage C:
- app/ai/engine.py            — AI proposal schema
- app/validation/decision.py  — deterministic decision validator
- app/risk/engine.py          — deterministic risk engine with sizing
- app/portfolio/controller.py — deterministic portfolio engine

## Test count
287 (A + B + C1 + C2) + 22 (C3) = 309 tests. CI green on every commit.

## Known issues
- Authorization object not yet implemented. C4.
- Correlation-adjusted portfolio limits deferred to Stage F (needs price history).
- Session verification deferred to historical qualification gate.
- CI dependencies unpinned. Stage F task.

## Open questions
- None.
