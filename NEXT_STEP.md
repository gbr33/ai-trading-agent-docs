# NEXT STEP

Stage E, Step 3 - ReplayEngine journal wiring.

Goal: the ReplayEngine writes a complete audit chain to the Journal on
every bar. Each bar is one transaction. Rows: market event, AI decision,
validation, risk, portfolio, orders, fills, trades. Uses deterministic
IDs derived from existing decision IDs so replay is reproducible. After
E3, every trade produced by a replay can be traced back to its market
event via trace_trade.

Awaiting: instructor to issue the step contract.
