# NEXT STEP

Stage B, Step 5 — Apply fills; modify_order; close_position.

Goal: wire the fill engine into SimulatedBroker. Add a method that takes a
market bar and processes every ACCEPTED or PARTIALLY_FILLED order through
try_fill, updating the order state machine and SimulatedAccount on each
result. Implement modify_order and close_position. Add SELL-side fill
support so close_position can use it.

Awaiting: instructor to issue the step contract.
