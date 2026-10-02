# NEXT STEP

Stage E, Step 2 - Trace query.

Goal: a query method that walks the audit spine from a trade_id back to
the original market event, returning every intermediate row in order:
trade -> exit fill -> entry fill -> order -> authorization ids ->
portfolio decision -> risk decision -> validation decision -> AI decision
-> market event. Blueprint Section 32.

Awaiting: instructor to issue the step contract.
