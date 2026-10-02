# PROJECT STATE

- Current stage: C — Common Decision Pipeline
- Last completed step: C1 — AI proposal schema and deterministic validator
- Next step: C2 — Risk engine
- Code repo last commit: f2b7999
- Docs repo last commit: (updated on push)

## Deliverables so far
Stage A:
- app/data/models.py, app/data/historical.py, app/data/quality.py, app/runtime/clock.py

Stage B:
- app/backtest/accounting.py, app/backtest/fills.py
- app/execution/state_machine.py, app/execution/broker.py, app/execution/simulated.py

Stage C:
- app/ai/engine.py          — AI proposal schema
- app/validation/decision.py — deterministic decision validator

## Test count
235 (A + B) + 29 (C1) = 264 tests. CI green on every commit.

## Known issues
- Risk engine not yet implemented. C2.
- Portfolio engine not yet implemented. C3.
- Authorization object not yet implemented. C4.
- Session verification deferred to historical qualification gate.
- CI dependencies unpinned. Stage F task.

## Open questions
- None.
