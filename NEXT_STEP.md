# NEXT STEP

Stage C, Step 1 — Shared validator.

Goal: a deterministic DecisionValidator that sits between AI output and 
the
risk engine. It checks schema, action, symbol, confidence threshold,
opportunity threshold, risk flags, entry/stop/target validity, session 
state,
and data freshness. A failed validation means NO ORDER. The validator 
never
modifies the risk policy to accommodate AI. Blueprint Sections 17, 19.

Awaiting: instructor to issue the step contract.
