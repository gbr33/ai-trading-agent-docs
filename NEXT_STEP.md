# NEXT STEP

Stage D, Step 1 — Historical event loop.

Goal: an event-driven replay engine that pulls MarketEvents one at a time
from a PointInTimeEventStream, advances the SimulationClock, and drives the
existing decision pipeline (validator -> risk -> portfolio -> authorization
-> broker) against a SimulatedBroker. No scanner, no AI, no news yet. This
step proves the loop works end to end on a hand-built scenario.

Also closes the C4 known issue: the orchestrator uses execute_authorized
exclusively, and submit_order becomes internal in a later D step.

Awaiting: instructor to issue the step contract.
