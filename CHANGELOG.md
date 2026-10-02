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

- B2 complete: immutable order model and validated state machine
  (code commit 8d14164). Also fixed a real bug: Pydantic v2 coerced bool 
to
  int before field validators ran, so `quantity=True` slipped past 
validation
  in both Position and SimulatedOrder. Fixed with Annotated[int, 
Field(strict=True)].
  Regression test added.

- B3 complete: abstract Broker interface and SimulatedBroker 
implementation
  (code commit 5c578a1). Order lifecycle only; no fills yet.

- B4 complete: fill engine with spread, slippage, commission, partial 
fills,
  and next-bar-only timing (code commit c69ec9b).

- B5 complete: apply fills in broker, add modify_order, close_position,
  SELL-side fills (code commit 4d331e1). Fixed a real bug: try_fill 
rejected
  PARTIALLY_FILLED orders, blocking continuation fills across bars. Now
  accepts both ACCEPTED and PARTIALLY_FILLED.
- Stage B complete: simulated broker fully functional end to end.

- C1 complete: AI proposal schema and deterministic decision validator
  (code commit f2b7999).

- C2 complete: deterministic risk engine with sizing and exposure caps (code commit 005cfdf).

- C3 complete: deterministic portfolio engine with combined and pending exposure (code commit 7400aef).

- C4 complete: ExecutionAuthorization object and authorized broker path (code commit 3ffe76b).
- Stage C complete: validator, risk, portfolio, and authorization. The common decision pipeline is done.

- D1 complete: event-driven replay engine driving the full decision pipeline end to end (code commit 7c10273).
\n- D2 complete: deterministic technical feature engine (code commit 04cf42e).\n\n- D3 complete: deterministic opportunity scanner (code commit 46561eb).\n\n- D4 complete: deterministic market regime classifier (code commit e3c2e9e).\n\n- D5 complete: deterministic point-in-time news engine (code commit b78921a).\n
- D6 complete: AI provider abstraction, heuristic provider, and fail-safe propose_safely wrapper (code commit 2e16705).

- D7 complete: strategy pipeline composing features, regime, news, scanner, and AI (code commit 15a9c22).

- D8 complete: position manager tracking exit parameters per symbol (code commit b264cb3).
