# PROJECT STATE

- Current stage: F - Historical Qualification
- Last completed step: F6 - Cost stress and dispersion analysis
- Next step: F7 - Parameter sensitivity (decision pending)
- Code repo last commit: (updated on push)
- Docs repo last commit: (updated on push)

## Stage F progress
- F1 OK CSV loader, IBKR downloader
- F2 OK One-day replay
- F3 OK One-week replay
- F4 OK One-month replay
- F5 OK Multi-month (Q3 2026)
- F6 OK Cost stress and dispersion analysis
- F7    Parameter sensitivity (decision pending)
- F8    Out-of-sample
- F9    Walk-forward
- F10   Benchmark comparison

## F6 results (Q3 2026, 64 sessions)

Dispersion analysis:
- 23 trades total, +390.65 zero friction
- After removing top 3 trades: +51.38
- After removing top 5 trades: -95.48
- Realized win/loss ratio: 1.76 (not 2.0)

Cost stress:
| friction  | trades | W  | L  | win% | total P&L |
|-----------|--------|----|----|------|-----------|
| zero      | 23     | 12 | 11 | 52.2 | +390.65   |
| normal    | 23     | 12 | 11 | 52.2 | -22.08    |
| moderate  | 23     | 8  | 15 | 34.8 | -639.94   |
| high      | 18     | 2  | 16 | 11.1 | -1359.22  |

Determinism: zero row reproduces F5 exactly.

## Test count
~870 tests. CI green on every commit.

## Known issues
- The strategy as configured does not survive realistic transaction
  costs. Blueprint Section 42 says do not advance.
- Return is concentrated. Removing top 3 trades reduces P&L from
  +390.65 to +51.38.
- Exit friction was previously missing; now corrected.
- Fill-price guards now prevent mis-anchored entries (Fix A).
- Structural coupling between heuristic confidence and validator
  minimum remains live.
- Exit reason not stored in trades table.

## Open questions
- Is there a robust region in the parameter space? F7.
- If F7 shows nothing, does the strategy need fundamental redesign?
- Or does the whole project continue on a different symbol/timeframe?
