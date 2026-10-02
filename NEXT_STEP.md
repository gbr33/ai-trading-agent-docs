# NEXT STEP

Stage D, Step 9 — Exit management.

Goal: an ExitEngine that inspects open positions on every bar and decides
whether to exit. Triggers: stop hit, target hit, time exit, end-of-day
flatten. Uses bar high/low conservatively when both stop and target fall
inside the same bar (Blueprint Section 28). Sells through the broker using
a synthesized execution authorization. Does not require a fresh AI call.

Awaiting: instructor to issue the step contract.
