# NEXT STEP

Stage D, Step 7 — Strategy module wiring into ReplayEngine.

Goal: a Strategy module that composes feature engine -> scanner -> regime
-> news -> AI provider -> ProposalBundle, and plugs into the ReplayEngine
strategy callback. It maintains a rolling per-symbol bar history so the
feature and regime engines can be called. It has no side effects and does
not touch the broker.

Awaiting: instructor to issue the step contract.
