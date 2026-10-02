# PROJECT STATE

- Current stage: C — Common Decision Pipeline
- Last completed step: C2 — Risk engine
- Next step: C3 — Portfolio engine
- Code repo last commit: 005cfdf
- Docs repo last commit: (updated on push)

## Deliverables so far
Stage A:
- app/data/models.py, app/data/historical.py, app/data/quality.py, app/runtime/clock.py

Stage B:
- app/backtest/accounting.py, app/backtest/fills.py
- app/execution/state_machine.py, app/execution/broker.py, app/execution/simulated.py

Stage C:
- app/ai/engine.py           — AI proposal schema
- app/validation/decision.py — deterministic decision validator
- app/risk/engine.py         — deterministic risk engine with sizing

## Test count
264 (A + B + C1) + 23 (C2) = 287 tests. CI green on every commit.

## Known issues
- Portfolio engine not yet implemented. C3.
- Authorization object not yet implemented. C4.
- Session verification deferred to historical qualification gate.
- CI dependencies unpinned. Stage F task.

## Open questions
- None.
