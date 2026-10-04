# NEXT STEP

Stage F, Step 7 - Parameter sensitivity.

Goal: test whether the strategy's behavior is robust across a range
of parameter values, or whether the zero-friction P&L lives at a
single lucky point. Under realistic costs the question changes: is
there any region where the strategy is profitable?

Options:
A. Small sweep near current defaults. RVOL threshold, RSI band,
   stop multiple, target multiple, confidence minimum.
B. Wide sweep to find any positive region.
C. Do not run F7. Accept the F6 result. Move to strategy redesign.

The honest read of F6 is that option C may be the right answer.
Section 42 is unambiguous. Running F7 risks discovering an
overfitted region that survives only in-sample.

Awaiting: user decision on A, B, or C.
