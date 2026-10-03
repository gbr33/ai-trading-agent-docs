# NEXT STEP

Stage F, Step 2 - One-day qualification replay.

Goal: run the ReplayEngine over the 390-bar AAPL dataset for 2026-09-08
and inspect the output. This is the first time the full pipeline
(features -> regime -> scanner -> news -> AI -> validation -> risk ->
portfolio -> authorization -> broker -> fills -> exits -> flatten ->
reconciliation -> journal) sees real market data.

Expected first-run outcome: something does not work the way we assumed.
That is normal. Blueprint Section 62 says: test -> inspect -> fix ->
repeat. Section 63 says the engine is not qualified merely because it
produces a report; it must demonstrate correctness first.

Awaiting: instructor to issue the step contract.
