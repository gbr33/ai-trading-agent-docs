# PROJECT STATE

- Current stage: F complete; moving to a new hypothesis (Option 2)
- Code repo last commit: e5f71a2
- Docs repo last commit: (updated on push)

## Stages complete
- A — Simulation foundation
- B — Simulated broker
- C — Common decision pipeline
- D — Complete replay
- E — Audit and journal
- F — Historical qualification (both strategy families falsified)

## Stage F outcomes
- Heuristic momentum screen: falsified (normal friction -22.08 on Q3)
- ORB v1: falsified out of sample (Q2 normal friction -303.79)
- ORB v2: falsified in sample (Q2 normal friction -623.09)

## Next
Option 2. New hypothesis. Overnight gap fade proposed. Pre-
registration pending. See NEXT_STEP.md.

## Test count
~880 tests. CI green on every commit.

## Known issues
- No surviving strategy after realistic friction.
- Structural coupling: heuristic confidence == opportunity score,
  validator needs 0.70. Documented, not fixed. Moot if heuristic
  path is unused.
- IBKR data unadjusted.
- CI dependencies unpinned.
