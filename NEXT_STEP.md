# NEXT STEP

Stage D, Step 8 — Position management.

Goal: a PositionTracker that maintains a per-symbol PositionPlan (stop,
target, entry_time, decision_id, entry order id) for every open position.
The tracker is informed when the broker fills an entry. It exposes lookup
by symbol, iteration, and removal when a position closes. It does not
trigger exits itself. Exit triggers are D9.

Awaiting: instructor to issue the step contract.
