# PROJECT STATE

- Current stage: F - Historical Qualification
- Last completed step: E6 - Experiment manifest (Stage E complete)
- Next step: F1 - Historical data preparation
- Code repo last commit: c3a4f4f
- Docs repo last commit: (updated on push)

## Stages complete
- A OK Simulation foundation
- B OK Simulated broker
- C OK Common decision pipeline
- D OK Complete replay
- E OK Audit and journal

## Stage F progress
- F1    Historical data preparation
- F2    One-day qualification
- F3    One-week qualification
- F4    One-month qualification
- F5    Multi-month / multi-year
- F6    Cost stress testing
- F7    Parameter sensitivity
- F8    Out-of-sample testing
- F9    Walk-forward testing
- F10   Benchmark comparison

## Test count
~750 tests across the codebase. CI green on every commit.

## Known issues
- submit_order remains public; execute_authorized is the production path.
- Regime classifier does not emit RISK_ON or RISK_OFF.
- AI provider is only the heuristic or a fixed provider. No real LLM.
- Session verification deferred to F1.
- CI dependencies unpinned. F task.
- Correlation-adjusted portfolio limits deferred to F7/F8.

## Open questions
- Where does historical data come from? (Vendor, format, cost.)
- What universe? How many symbols?
- What timeframe? 1-minute is the assumed default.
- What date range for the first qualification?

## The next conversation
The user must decide on data source, universe, timeframe, and date range
before F1 can start. See NEXT_STEP.md.
