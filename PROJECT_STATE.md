# PROJECT STATE

- Current stage: B — Simulated Broker
- Last completed step: B1 — Simulated account
- Next step: B2 — Simulated positions (order-level position updates)
- Code repo last commit: 3524511
- Docs repo last commit: (updated on push)

## Stage A deliverables (all present, all tested)
- app/data/models.py       — MarketEvent, DatasetManifest, enums
- app/data/historical.py   — HistoricalDataProvider, 
PointInTimeEventStream
- app/data/quality.py      — fail-closed dataset validator
- app/runtime/clock.py     — simulation clock, session derivation

## Stage B deliverables so far
- app/backtest/accounting.py — SimulatedAccount + Position (long-only)

## Test count
79 (Stage A) + 24 (B1) = 103 tests. CI green on every commit.

## Known issues
- Session verification (each event's session vs. exchange calendar) is 
deferred
  to the historical qualification gate (Blueprint Section 63).
- Position object is created by accounting; it will be reused by the order
  manager in B2 without change.
- All Python dependencies in CI are unpinned. Pin exact versions in Stage 
F
  for reproducible qualification runs.

## Open questions
- None.
