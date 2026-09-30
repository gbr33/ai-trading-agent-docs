# PROJECT STATE

- Current stage: B — Simulated Broker
- Last completed step: B3 — Simulated broker
- Next step: B4 — Fill engine
- Code repo last commit: 5c578a1
- Docs repo last commit: (updated on push)

## Stage A deliverables
- app/data/models.py       — MarketEvent, DatasetManifest, enums
- app/data/historical.py   — HistoricalDataProvider, 
PointInTimeEventStream
- app/data/quality.py      — fail-closed dataset validator
- app/runtime/clock.py     — simulation clock, session derivation

## Stage B deliverables so far
- app/backtest/accounting.py      — SimulatedAccount + Position 
(long-only)
- app/execution/state_machine.py  — SimulatedOrder + transition graph
- app/execution/broker.py         — abstract Broker interface + typed 
exceptions
- app/execution/simulated.py      — SimulatedBroker implementation

## Test count
79 (Stage A) + 24 (B1) + 1 (B1 regression) + 40 (B2) + 35 (B3) = 179 
tests.
CI green on every commit.

## Known issues
- Authorization check (Blueprint Section 60 Invariant 2: "no order without
  authorization") is deferred to Stage C. The broker currently accepts any
  structurally valid order.
- modify_order and close_position raise NotImplementedError. Deferred to 
B4
  (fill engine).
- Session verification (event session vs. calendar) deferred to the 
historical
  qualification gate.
- All Python dependencies in CI are unpinned. Pinning is a Stage F task.

## Open questions
- None.
