# NEXT STEP

Stage B, Step 2 — Simulated orders.

Goal: a SimulatedOrder model and order state machine (CREATED, SUBMITTED,
ACCEPTED, PARTIALLY_FILLED, FILLED, CANCELLED, REJECTED) that the 
simulated
broker will drive. Orders are immutable value objects; state transitions 
are
validated against the allowed graph. No code path assumes a fill just 
because
submission succeeded.

Awaiting: instructor to issue the step contract.
