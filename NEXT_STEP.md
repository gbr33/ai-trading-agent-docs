# NEXT STEP

Stage C, Step 2 — Risk engine.

Goal: a deterministic RiskEngine that is the highest authority over trade
admission. Evaluates account-level risk (equity, daily loss), trade-level
risk (entry, stop, target, risk per share, max dollar risk), position limits
(max open, max exposure, symbol, sector), and activity limits (max daily
trades, cooldown, entry cutoff). Computes position size from stop distance.
Returns a RiskDecision. Blueprint Sections 20, 21.

Awaiting: instructor to issue the step contract.
