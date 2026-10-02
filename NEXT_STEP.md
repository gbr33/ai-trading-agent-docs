# NEXT STEP

Stage D, Step 11 - Reconciliation.

Goal: a reconciliation engine that compares internal account state
(SimulatedAccount positions) against broker-reported state (broker
get_positions) and detects any mismatch. In the simulated broker these
should always agree; the engine is a structural check that will catch
regressions and will be required for Stage G paper trading. Returns a
structured ReconciliationResult. Blueprint Section 51.

Awaiting: instructor to issue the step contract.
