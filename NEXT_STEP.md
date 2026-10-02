# NEXT STEP

Stage E, Step 4 - Trades and flat closes.

Goal: write trades rows when a position exits (via exit engine or flat
engine). Requires ExitResult to carry entry_fill_id and quantity so the
trade can be linked to its entry. Flat engine needs to return per-symbol
closed positions with fill info instead of just a FlattenResult summary.
Then trace_trade can walk all the way from a closed trade back to its
market event with no missing links.

Awaiting: instructor to issue the step contract.
