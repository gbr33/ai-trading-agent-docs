## 2026-09-30
- A1 complete: canonical MarketEvent with structural validators (code 
commit 16bc51f).
- A2 complete: simulation clock with session derivation from XNYS calendar 
(code commit f73975d).
- A3 complete: historical data provider with fail-closed ordering (code 
commit 03fd0ca).
- A4 complete: dataset manifest and fail-closed dataset validator (code 
commit f728c55).
- A5 complete: point-in-time event stream enforcing anti-lookahead (code 
commit 245cdc2).
- Stage A complete: simulation foundation — canonical market event, 
simulation
  clock, historical data provider, dataset validator, point-in-time 
boundary.
  All A-stage tests passing.

- B1 complete: simulated account and position, long-only, Decimal-correct
  (code commit 3524511). CI green.
- ci: GitHub Actions workflow running ruff, mypy, pytest on every push.
