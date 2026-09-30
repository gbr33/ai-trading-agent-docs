# PROJECT STATE

- Current stage: B — Simulated Broker
- Last completed step: B2 — Simulated orders (order model + state machine)
- Next step: B3 — Simulated broker
- Code repo last commit: 8d14164
- Docs repo last commit: (updated on push)

## Stage A deliverables
- app/data/models.py       — MarketEvent, DatasetManifest, enums
- app/data/historical.py   — HistoricalDataProvider, 
PointInTimeEventStream
- app/data/quality.py      — fail-closed dataset validator
- app/runtime/clock.py     — simulation clock, session derivation

## Stage B deliverables so far
- app/backtest/accounting.py     — SimulatedAccount + Position (long-only)
- app/execution/state_machine.py — SimulatedOrder + transition graph

## Test count
79 (Stage A) + 24 (B1) + 40 (B2) + 1 (B1 regression for bool) = 144 tests.
CI green on every commit.

## Known issues
- Session verification (event session vs. calendar) deferred to historical
  qualification gate.
- All Python dependencies in CI are unpinned. Pinning is a Stage F task.

## Open questions
- None.
