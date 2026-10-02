# NEXT STEP

Stage D, Step 6 — AI integration.

Goal: (a) an AI client abstraction in app/ai/engine.py that turns a
structured context (symbol, features, regime, news, opportunity, portfolio
context) into an AIProposal via a pluggable provider; (b) a concrete
HeuristicProvider for deterministic testing with no external API; (c) a
strategy module that composes scanner -> regime -> news -> AI ->
ProposalBundle and plugs into the ReplayEngine callback.

Fail-safe behavior: any provider error, timeout, malformed output, or
schema violation yields HOLD. No provider has execution authority.

Awaiting: instructor to issue the step contract.
