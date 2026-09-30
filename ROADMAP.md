
---

## Deferred work — AI skills and tools

Items from the earlier "AI skills and tools" checklist. None of these 
start
before the stage listed. Do not pull them forward.

### AI skills (Blueprint Sections 17, 18, 19, 79)
All live in `app/ai/engine.py` and `app/validation/decision.py` unless 
noted.

- [ ] Structured JSON output enforced by schema — Stage C
- [ ] Schema validation rejecting malformed AI output — Stage C
- [ ] Hallucination checks (numbers, symbols, timestamps match inputs) — 
Stage C
- [ ] Risk-flag extraction from AI output — Stage C
- [ ] Thesis and invalidation-condition extraction — Stage C
- [ ] News summarization into structured events — Stage D
- [ ] Sentiment scoring with source attribution — Stage D
- [ ] Prompt injection defense in news pipeline — Stage D
- [ ] Timeout → HOLD/REJECT fallback — Stage D
- [ ] Cost and latency budgets per call and per day — Stage D
- [ ] Offline evaluation harness for prompt/model changes — Stage D
- [ ] Recorded AI decision replay — Stage E
- [ ] Model version pinning per decision — Stage E
- [ ] Confidence calibration tracking — Stage E + research
- [ ] Human review gate before prompt/model change reaches live — Stage G

Rule: every item above is a filter or a recorder, never an executor.
The AI never gets a path to the broker.

### Engineering tools (Blueprint Sections 81, 79)
- [ ] hypothesis property-based tests on risk and sizing invariants — 
Stage F
- [ ] structlog JSON logging in runtime and monitoring — Stage D
- [ ] SQLite journal database in `app/journal/database.py` — Stage E
- [ ] GitHub Actions CI: ruff, mypy, pytest on every push — after Stage B 
Step 1
- [ ] pre-commit hooks (ruff, mypy) — after Stage B Step 1
- [ ] Docker — deferred. Not needed on Intel MacBookPro13,1 (8 GB RAM).
- [ ] 1Password CLI or equivalent secret manager — Stage G (before paper)

### Data and ops tools (Blueprint Sections 74–78)
- [ ] Point-in-time universe and symbol master — Stage D/E
- [ ] Corporate action feed (splits, dividends, symbol changes) — Stage F
- [ ] Survivorship correction / delisted symbols — Stage F
- [ ] Halt feed (LULD, news, circuit breaker) — Stage F/G
- [ ] NTP sync verification — already handled by macOS, verify before 
paper
- [ ] Process supervisor (supervisor package on macOS) — Stage G
- [ ] Backup and restore drill for journal DB — Stage E
- [ ] Discord alert channel webhook wiring — Stage D/E
- [ ] PROJECT_STATE.md and NEXT_STEP.md discipline — **in use now**

### Live-safety reminder
None of the above permit live trading. Section 82 gate still applies.
