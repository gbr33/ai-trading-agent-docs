## 2026-09-30 — A3 design decisions
Historical data provider:

1. Fail closed on out-of-order input. Provider raises ValueError at 
construction
   if timestamps are not non-decreasing. It does not silently sort. Silent 
sorting
   would hide dataset corruption. Blueprint Sections 9, 34, 58.

2. Streaming, one event at a time. No peek, no random access, no 
events_until(t).
   The engine pulls; the provider yields. The safest anti-lookahead 
interface is
   one that cannot express future access. Blueprint Section 8.

3. Non-decreasing, not strictly increasing. Two events at the same 
timestamp for
   different symbols are allowed. Duplicate detection is a data-quality 
concern
   (Section 12), not a provider concern.

4. In-memory for now. The provider accepts any Iterable[MarketEvent]. A 
CSV or
   file loader will feed it in Step A4.

## 2026-09-30 — A4 design decisions
Historical dataset validation:

1. Blocking vs warning separation. Blocking issues reject the dataset 
entirely.
   Warnings are recorded but the dataset still validates. An illiquid 
symbol may
   legitimately have missing bars; a duplicate bar is corruption.
   Blueprint Sections 9, 12, 38, 58.

2. Manifest as a Pydantic model. A manifest is a data structure (Section 
38),
   not validation logic. It belongs in app/data/models.py alongside 
MarketEvent.

3. Fail closed. Any blocking issue sets is_valid = False. The replay 
engine
   must refuse to run on an invalid dataset.

4. Gap detection scoped to intra-session, same-day, same-symbol pairs. 
Overnight,
   weekend, and holiday gaps are legitimate and must not warn.
   Blueprint Sections 11, 38.

5. Gap threshold = 2x expected interval. A single missing 1-minute bar 
produces
   a 2-minute delta. Threshold is a module-level constant.

6. Session verification deferred. Checking each event's session field 
against
   the exchange calendar is a separate concern, added later. Recorded as a 
known
   limitation in PROJECT_STATE.md.

## 2026-09-30 — Process change for A5 onward
Ruff import-order errors appeared in every step from A1 to A4, each 
requiring
a follow-up "fix ruff import order" commit. Starting with A5, new files 
will be
written with imports pre-sorted (stdlib, then third-party, then 
first-party,
alphabetized within each group). If a step still produces a ruff failure, 
the
commit does not land — the error is pasted and corrected first.

## 2026-09-30 — A5 design decisions
Point-in-time event stream:

1. Pull-based only. No iteration over the stream, no peek, no random 
access.
   The safest anti-lookahead interface cannot express future access.
   Blueprint Sections 8, 9.

2. Internal buffer, never exposed. Future events pulled from the provider 
are
   held privately and released only after the clock reaches their 
timestamp.

3. Clock is the sole authority. The stream reads clock.now and never calls
   datetime.now(). Blueprint Section 10.

4. Backward clock moves still raise. advance_to delegates to 
SimulationClock,
   which already rejects backward motion.

5. Anti-lookahead tests included. Two tests in tests/test_point_in_time.py
   enforce Section 37 directly: a future event never appears in released
   output, and dataset truncation produces identical behavior on the 
retained
   prefix.

## 2026-09-30 — Process change for Stage B onward
Ruff import-order errors appeared in every A-stage step and required a
follow-up "fix ruff import order" commit each time. Starting with Stage B,
every step runs `ruff check --fix` before the manual `ruff check`. Safe 
fixes
are applied first, then verified. If the second check is not clean, we 
stop
and fix manually before committing.

## 2026-09-30 — B1 design decisions
Simulated account:

1. Decimal for all money. Cash, prices, P&L, commissions, slippage. 
Blueprint
   Section 29 requires deterministic account-state changes.

2. Position is frozen; SimulatedAccount is mutable. Same pattern as
   MarketEvent. Updating a position replaces the object.

3. Long-only for B1. Quantity > 0 enforced. Shorts are a later stage.

4. One position per symbol. Adding to an existing position adjusts the 
average
   entry price, quantized to 6 decimal places. Partial close leaves the
   average entry price unchanged.

5. Fail closed on overdraft. Open that would overdraw cash raises before 
any
   state change. Public methods are atomic.

6. Slippage is a reporting metric, not a cash movement. It is already 
reflected
   in the fill price. The account accumulates it for later analysis.

7. No clock inside the account. All timestamps are passed by the caller.

8. Bool rejected as quantity. Python treats True as 1. A bool quantity is 
a bug
   and must fail.

## 2026-09-30 — Process change confirmed
Ruff --fix now runs first in every step. Of 87 errors reported on B1, 86 
were
auto-fixed (mostly import order and formatting); the 1 real rule violation
(TRY004, ValueError vs TypeError) was fixed manually. This eliminated the
follow-up "fix ruff import order" commits that occurred on every A-stage 
step.

## 2026-09-30 — CI live
GitHub Actions runs ruff, mypy, pytest on every push to main of
gbr33/ai-trading-agent (private). First code commit through CI: 3524511.
Actions updated to Node 24 compatible versions (checkout@v5, 
setup-python@v6).

## 2026-09-30 — B2 design decisions
Order model and state machine:

1. Frozen SimulatedOrder. Transitions return new objects via 
model_validate.
   Same pattern as MarketEvent and Position.

2. Explicit transition graph in ALLOWED_TRANSITIONS dict. No if/else 
chains.
   Blueprint Section 26.

3. Terminal states (FILLED, CANCELLED, REJECTED) have no outgoing 
transitions.

4. No shortcut transitions. CREATED -> ACCEPTED is rejected. Broker path 
is
   CREATED -> SUBMITTED -> ACCEPTED. Blueprint Section 26.

5. Partial fills stay in PARTIALLY_FILLED until filled_quantity == 
quantity.
   Average fill price is a quantity-weighted average, quantized to 6 dp.
   Blueprint Sections 26 and 78.

6. Strict int fields via Annotated[int, Field(strict=True)]. Prevents 
Python's
   True == 1 coercion from bypassing field validators. Regression test 
added.

7. order_id is caller-supplied. No UUID generation. Required for 
deterministic
   replay. Blueprint Section 36.

8. No broker logic in state_machine.py. No cash. No sessions. No risk. 
Pure
   transition validation. Separates concerns.

## 2026-09-30 — Lesson: Pydantic v2 bool coercion
Pydantic v2 coerces bool to int by default before field validators run. A
field validator checking `isinstance(v, bool)` sees the coerced int, not 
the
bool. Annotated[int, Field(strict=True)] is the correct fix. This bug was
caught by tests in B2 and fixed in both the new SimulatedOrder and the 
existing
Position. Lesson recorded: validate integer business fields with strict 
mode,
not isinstance checks inside field validators.

## 2026-09-30 — B3 design decisions
Broker interface and simulated broker:

1. Abstract base class enforces the interface. Every broker (simulated,
   paper, IBKR) subclasses Broker. The strategy never knows which is 
active.
   Blueprint Section 24.

2. Typed exceptions: BrokerError base, BrokerNotConnected, OrderNotFound,
   OrderRejected. Callers catch categories, not bare Exception.

3. Account injected, not constructed. SimulatedBroker(account). Tests and
   future stages can substitute any SimulatedAccount.

4. Connection gate on every operation except connect, disconnect,
   is_connected. Same behavior a real broker API has.

5. submit_order walks CREATED -> SUBMITTED -> ACCEPTED in one call, or
   returns a REJECTED order. Callers must check the returned order; a
   successful call does not imply acceptance. Blueprint Section 26.

6. submit does not move cash or create positions. Those are fill-engine
   concerns (B4).

7. modify_order and close_position raise NotImplementedError. Explicit,
   fail-closed. No silent no-op.

8. cancel_all returns the list of newly-cancelled orders for audit.

9. Authorization deferred to Stage C. Documented as a known limitation.
   Blueprint Section 60 Invariant 2 will be enforced when the 
authorization
   object exists.

## 2026-09-30 — B4 design decisions
Fill engine:

1. Pure function, no side effects. try_fill returns FillResult | None and
   never touches the account or the order. The broker applies fills in B5.
   Makes the engine trivial to test and deterministic.

2. Next-bar fills only. bar.timestamp must be strictly after
   order.created_at. Same-bar fills are lookahead and are forbidden.
   Blueprint Sections 8, 9, 37.

3. Half-spread against trade direction. BUY fill = P * (1 + 
spread_bps/2/10000).
   Represents paying the ask when buying. Counterpart for SELL in B5.

4. Slippage is separate from spread so cost stress can vary one without 
the
   other. Blueprint Section 42.

5. Limit fill price is best-of. For BUY LIMIT at L: if bar.open < L, fill 
at
   bar.open. Otherwise fill at L. Matches real limit order behavior.

6. Partial fills from bar volume.
   max_fillable = floor(bar.volume * max_fill_pct_of_bar_volume).
   Blueprint Sections 25, 27.

7. Commission = max(commission_min, commission_per_share * fill_qty).
   Simple, auditable. Real brokers have tiers; those can be layered later
   without changing the interface.

8. Decimal throughout. Fill price quantized to 6 dp.

9. BUY only for B4. SELL-side deferred to B5 with exits.

## 2026-09-30 — B5 design decisions
Apply fills; modify_order; close_position; SELL fills:

1. Broker is the only place fills are applied. try_fill stays pure.
   process_bar orchestrates: pull working orders, try each, apply.
   Blueprint Sections 25, 27.

2. Fill model injected at construction. Default is zero friction. The
   backtest engine will supply the configured model.

3. SELL price direction is symmetric to BUY. Sell at bid, accept worse
   prices. LIMIT SELL caps at the limit; we never accept less.

4. SELL fills require an existing position. If account.close_position
   raises, the order is CANCELLED and no account state changes. Fail
   closed. Blueprint Section 58.

5. No shared volume budget within a bar. Documented simplification.

6. modify_order is not cancel + new. It updates the existing order in
   place. Preserves the audit thread. Cannot change order type via modify.

7. close_position bypasses fill friction. The caller supplies the price.
   Used by flatten engine and manual exits.

8. process_bar returns the list of applied fills for journaling.
   Blueprint Section 31.

9. Order iteration in insertion order. Deterministic. Blueprint Section 
36.

## 2026-09-30 — Bug: try_fill rejected PARTIALLY_FILLED
The first version of try_fill accepted only ACCEPTED orders. A partial 
fill
moved the order to PARTIALLY_FILLED, and the next bar's try_fill raised
ValueError instead of continuing to fill. This broke multi-bar fills 
entirely.
Caught by test_process_bar_partial_then_full_on_next_bar. Fixed: try_fill 
now
accepts both ACCEPTED and PARTIALLY_FILLED. Lesson: when a status 
transition
enables a repeatable operation (filling), the gate must include every 
state
that operation can run in, not just the initial state.

## 2026-09-30 — Lesson: scripted multi-file edits
The B5 correction step needed 8 coordinated edits across 3 files. Doing 
them
by hand in nano would have been error-prone. A single Python script with
explicit `assert` guards for each pattern is safer: it fails loudly if any
pattern does not match, so no partial state lands. Reuse this pattern for
future multi-file corrections.

## 2026-10-02 — C1 design decisions
AI proposal schema and validator:

1. AIProposal is frozen, extra fields rejected. Blueprint Section 17.
2. Validator is a pure function. No side effects. Failures are data, not
   exceptions.
3. HOLD short-circuits and approves as a no-op.
4. All non-HOLD checks always run. The result carries every check for the
   audit trail.
5. AI does not propose prices. Entry, stop, target come from the strategy
   layer. The validator sees both and validates the combination.
6. ValidationConfig is frozen and injected. No hidden thresholds.
7. Data freshness is symmetric (abs of the delta). Catches lookahead and
   stale data.
8. Any quality flag on the bar vetoes the decision.
9. blocking_risk_flags is config-supplied. The AI does not decide what is
   dangerous.
10. No journal write in this step. The validator returns a result; recording
    is a later concern.

## 2026-10-02 — Lesson: regex field edits need trailing-comma care
The regex replacement for `entry: "object"` -> `entry: Decimal` left a
trailing comma, producing invalid Python. Regex edits on class field
declarations must account for the fact that the last field in a block may
have no comma and that class bodies do not accept trailing commas after a
bare annotation. Fix was a second pass to strip the commas. Lesson: after
any automated multi-line edit, run ruff immediately and treat syntax errors
as expected failure modes, not surprises.

## 2026-10-02 — C2 design decisions
Risk engine:

1. Pure function. evaluate_risk returns a RiskDecision. No side effects.

2. Failures are data, not exceptions. Same pattern as the validator.

3. One risk policy for sim, paper, live. This module is the single source
   of truth. No stage-specific variants. Blueprint Section 68.

4. Sizing is deterministic and integer. floor on shares. Blueprint Section 21.

5. Exposure caps are fractions of equity. new_position_value / equity <=
   cap. Absolute dollar caps are a Stage F refinement.

6. Equity is the sizing base, not cash. Cash sufficiency is the broker''s
   job at fill time.

7. Sector map is config-supplied. Symbols not in the map default to
   UNKNOWN. UNKNOWN still counts toward total exposure.

8. Adds to existing positions are allowed. The open-positions check is
   bypassed when the symbol is already held. Sizing still applies all
   exposure caps.

9. Cooldown is optional. If last_trade_time is None or cooldown is 0,
   the check passes.

10. Sizing reduction is the last step. Checks 1-6 and 9-12 run against the
    raw sized quantity. Exposure caps may reduce the final size. If
    reduction drives size to 0, the position_size_above_zero check fails
    and the decision is rejected.

## 2026-10-02 — Lesson: nested test with default config
test_basic_sizing_from_stop_distance expected 1000 shares but the default
config caps symbol exposure at 30% of equity (300 shares). The engine was
correct; the test setup was wrong. Lesson: tests that exercise one specific
mechanism must neutralize unrelated config values, or they silently test
something else.

## 2026-10-02 — C3 design decisions
Portfolio engine:

1. Pure function. Same pattern as validator and risk engine.

2. Pending order awareness is the key value-add over the risk engine.
   Risk sees filled positions; portfolio sees filled plus working.

3. Correlation is deferred honestly. Without price history in this module,
   any correlation number would be fabricated. Stage F adds it with real
   price data. Documented as a known issue.

4. Pending SELL orders offset net long exposure. Prevents the portfolio
   engine from rejecting a new BUY when the trader is about to exit.

5. Capital check at portfolio level, not risk level. Risk sizes on equity;
   portfolio verifies cash covers the sized quantity plus outstanding BUY
   obligations. First line of defense against overcommitment.

6. Size reduction is the last step. Same pattern as the risk engine.

7. Duplicate pending symbol rejected by default. Config can allow it.
   Rationale: two working orders on the same symbol are almost always a
   caller bug.

8. Engine and its types live in controller.py, matching Blueprint Section 5.

## 2026-10-02 — C4 design decisions
Execution authorization:

1. Two paths, honestly named. submit_order is the low-level primitive kept
   for tests and internal use. execute_authorized is the production path
   that requires a valid authorization. The invariant "no order without
   authorization" is enforced on the production path. Closing the gap on
   submit_order is a Stage D task when the orchestrator takes over.

2. IDs are content hashes, not UUIDs. sha256 truncated to 16 hex chars.
   Deterministic replay requires reproducible IDs. Blueprint Section 36.

3. quantity is the portfolio final_quantity when authorized, else 0. The
   portfolio engine already applied every reduction. No further reduction
   at authorization time.

4. exposure defaults to quantity * entry, overridable by the caller.

5. Fail closed. authorized is False if any of the three upstream decisions
   is not approved. execute_authorized refuses with BrokerError.

6. limit_price invariant matches the order state machine. MARKET forbids
   it, LIMIT requires it. Cross-field validated.

7. created_at is caller-supplied. No datetime.now() inside. Blueprint
   Section 10.

8. Authorization is not persisted by the broker. Persistence is Stage E.

## 2026-10-02 — Stage C complete
The common decision pipeline is done. From here forward, sim, paper, and
live will all use the same code: validator -> risk -> portfolio ->
authorization -> broker. Only the broker adapter changes. Blueprint
Sections 4 and 68 are now structural facts, not goals.

## 2026-10-02 — D1 design decisions
Historical replay engine:

1. Strategy is a callback, injected. No scanner, no AI, no news. The loop
   has one job: run the pipeline. D2 plugs in the real strategy without
   changing the loop.

2. The engine uses execute_authorized exclusively. submit_order stays
   public for tests. Closing that is a later D step.

3. Fill first, then decide. On each bar, process_bar runs before the
   strategy callback. The strategy sees post-fill account state. Realistic
   and deterministic.

4. Per-step record is the audit unit. ReplayStep captures everything from
   market event to order. Blueprint Section 30.

5. Fail-closed at each layer. Any rejection records a reason and no order
   is placed. Other bars proceed normally.

6. Deterministic. No randomness, no datetime.now(). All time from the
   clock. Blueprint Sections 10, 36.

7. Engine does not own clock or stream. Both are injected.

8. No journal, no persistence. ReplayResult is in memory. Stage E.

## 2026-10-02 — Lesson: actions in class body must end with comma
The D1 correction script replaced "bundle.proposal.action_to_side()" (a
method that did not exist) with a helper call. The regex dropped the
trailing comma, producing syntax errors at three call sites. Lesson:
when replacing a term inside a call-argument list, include the comma in
both the search and the replacement.

## 2026-10-02 — Lesson: test strategy must account for pipeline state
The D1 test_approved_proposal_places_order initially expected two
proposals to produce two orders. But process_bar runs before the strategy
callback, so on bar 2 the first order had already filled and the symbol
cap was exhausted. The engine correctly rejected the second BUY. The
test was wrong, not the engine. Lesson: tests that emit a proposal on
every bar must reason about post-fill portfolio state, or emit only once.

## 2026-10-02 — D2 design decisions
Feature engine:

1. Pure function: history-in, snapshot-out. Deterministic.

2. The current bar is the last element of history. No separate current
   parameter. Future bars are never seen.

3. None means insufficient history, not zero. Fail-closed. Callers must
   check. Blueprint Section 58.

4. RVOL excludes the current bar from the baseline. Blueprint Section 13.

5. Wilder RSI and Wilder ATR. Standard industry definitions.

6. RSI edge cases handled explicitly: all-gains -> 100, all-losses -> 0,
   flat -> 50.

7. All prices use Decimal. Ratios and returns use float.

8. Trend is relative to two SMAs, no hidden threshold.

9. Breakout is close-based, not intraday-high-based.

10. No caching, no incremental updates. Full recompute per bar. Stage F
    optimization if needed.

## 2026-10-02 — Lesson: test helper defaults must respect validators
The bar helper defaulted high=101, low=99, so any close above 101 (in
trend, momentum, and other tests) failed the MarketEvent OHLC validator.
Fix: derive high/low from close unless overridden. Lesson: when a helper
wraps a model with cross-field validators, its defaults must satisfy those
validators for all caller-supplied values.

## 2026-10-02 — Lesson: float variance inference
variance ** 0.5 infers as Any in mypy strict mode. Annotate the variance
expression as float and cast the final return. This is the pattern for any
numeric expression where the operand types are not visible to the checker.

## 2026-10-02 — D3 design decisions
Opportunity scanner:

1. Pure functions. scan_one and scan_many. Same input, same output.

2. Fail closed. Missing rvol, rsi, or momentum short-circuits to None.

3. Scanner filters and ranks. It does not select. Selection is a caller
   concern.

4. Configurable thresholds. RVOL > 3 and the RSI band are config values,
   not code constants. Blueprint Section 14.

5. Scoring weights sum to 1.0, enforced by config validation. Score
   remains in [0, 1].

6. Clamping on every component. One strong signal cannot dominate.

7. Reasons list is auditable. Journal in Stage E will persist it.

8. Deterministic tiebreak: (-score, symbol, timestamp). No insertion-order
   dependency.

9. require_breakout and require_trend are optional hard filters. Off by
   default so the scanner works as a pure ranker.

10. No _tz, no clock, no datetime.now(). Time comes from the snapshot.
    Blueprint Section 10.

## 2026-10-02 — D4 design decisions
Regime classifier:

1. Pure function. classify_regime(history, config). Same input, same output.

2. No AI, no news. RISK_ON and RISK_OFF are not implemented. They need
   breadth or news data we do not have. Documented, not faked.

3. Linear regression on closes, not on returns. Closes give a slope with a
   natural "fraction per bar" unit after normalization.

4. R-squared gates the trend label. High slope with low R-squared means
   noise with drift, not a trend.

5. Volatility takes precedence over trend. A high-volatility period is
   labeled HIGH_VOLATILITY even if a trend is visible.

6. Annualized volatility via sqrt(252). Standard convention.

7. UNKNOWN on insufficient data. Fail-closed. Blueprint Section 15.

8. Reasons list is auditable. Stage E will persist it.

9. Ordered check chain, first match wins. Deterministic, no weights.

10. No _tz, no clock, no datetime.now(). Time comes from input data.

## 2026-10-02 — Lesson: ruff ISC004 on multi-line f-strings in tuples
When a tuple contains an implicitly concatenated f-string, ruff ISC004
fires unless the whole concatenation is wrapped in parentheses. Fix:
wrap the concatenated parts in a nested parenthesized expression inside
the tuple.

## 2026-10-02 — D5 design decisions
News engine:

1. Pure function. records-in, snapshot-out. No external calls.

2. Point-in-time boundary requires BOTH news_timestamp <= as_of AND
   processing_timestamp <= as_of. Strictest interpretation. Prevents
   lookahead where an item was published earlier but ingested later.

3. processing_timestamp must be >= news_timestamp. Otherwise the record
   is malformed and is rejected at construction.

4. None means "unavailable", not "neutral". Snapshot with zero records
   has aggregate_sentiment=None.

5. Sentiment aggregation weighted by relevance * confidence.

6. max_age_seconds bounds the lookback. Prevents stale headlines from
   influencing decisions.

7. max_records_per_symbol bounds the snapshot. Keeps the AI prompt
   within size limits.

8. Sort by (-news_timestamp, news_id). Deterministic.

9. blocking_categories is config-supplied. Engine flags them; caller
   decides what to do.

10. No prompt injection defense at this layer. Structured records only.
    D6 (AI integration) is where untrusted-text handling matters.

## 2026-10-02 — Lesson: test premise must satisfy upstream filters
test_aggregate_sentiment_none_when_all_zero_weight initially set
relevance=0.0 to trigger the zero-weight aggregate branch, but
min_relevance=0.1 filtered the record out first. Fix: relevance=0.5
(passes filter) with confidence=0.0 (zeroes weight). Lesson: a test
targeting a downstream code path must first satisfy every upstream
filter.

## 2026-10-02 — D6 design decisions
AI provider abstraction:

1. Provider is an ABC. Same pattern as Broker. Concrete implementations
   in later stages (real LLM). Tests and D7 use HeuristicProvider.

2. No network in D6. Heuristic is deterministic and offline.

3. propose_safely is the only path the pipeline uses. Any exception from
   the provider is caught and converted to HOLD. Fail-safe by construction.
   Blueprint Section 79.

4. HOLD carries a reason in thesis. Journal records it. No silent no-ops.

5. risk_flags names blocking news categories. Validator's
   blocking_risk_flags controls whether they actually veto.

6. SELL is not produced in D6. The AI does not decide exits. Exit
   management is D9. Heuristic proposes BUY or HOLD only.

7. Portfolio context is read-only. Provider cannot reach the broker.

8. AIResponse wraps the proposal and adds provider metadata for replay.

9. No prompt construction in D6. Real-LLM concern for later stages.

## 2026-10-02 — Scope correction
NEXT_STEP.md after D5 grouped three concerns into D6. D6 is now AI
provider abstraction only. Strategy wiring becomes D7. Position
management, exit management, flattening, and reconciliation shift
accordingly.

## 2026-10-02 — D7 design decisions
Strategy pipeline:

1. Stateful only in per-symbol rolling history. Bounded by max_history_bars.

2. Returns None on HOLD or SELL. SELL is exit logic; deferred to D9.

3. ATR-based stop and target. Configurable multiples.

4. Does not construct prompts. AIRequest is the interface; prompt building
   is a real-LLM concern.

5. News records loaded at construction; filtered per bar for point-in-time.

6. Portfolio context read from EngineContext. Read-only.

7. Sector from sector_map with default UNKNOWN. Consistent with risk and
   portfolio engines.

8. No journal writes. No order placement. Pure proposal generation.

9. The blueprint does not list this file. It was approved by the user with
   the justification that the strategy composer is a distinct concern from
   the pure-function scanner and from the runtime orchestrator.

## 2026-10-02 — D8 design decisions
Position manager:

1. Stateful tracker, not a decision maker. Registers and looks up plans.
   Does not act. D9 owns exit logic.

2. PositionPlan is separate from account Position. Account tracks quantity
   and average price. Plan tracks stop, target, decision_id, entry_time,
   order_id.

3. Long-only. SELL entries rejected.

4. Weighted average entry price, quantized to 6 dp.

5. Same decision_id required for accumulation.

6. Insertion order preserved.

7. remove returns bool.

8. FillApplication carries a reason.

9. No datetime.now(). entry_time comes from the caller.

10. OrderManager does not import SimulatedBroker or SimulatedAccount.

## 2026-10-02 — Lesson: test cannot exercise unreachable guards
test_register_rejects_zero_fill tried to verify the zero-quantity guard
inside register_entry, but FillResult refuses qty=0 at construction, so
the guard is unreachable through normal call paths. Rewrote the test to
verify the outer contract.

## 2026-10-02 - D9 design decisions
Exit engine:

1. Separate from the broker. The broker executes; the engine decides.

2. Conservative ambiguous-bar rule. When both stop and target fall inside
   the same bar, assume stop first. Configurable; on by default. Blueprint
   Section 28.

3. Exits bypass the decision pipeline. No validator, no risk, no
   portfolio, no AI. A stop is a pre-authorized order.

4. close_position is the execution path. Reuses existing broker code.

5. Plan removal only after successful exit. If the broker raises, the plan
   stays for the next bar.

6. Priority order fixed: stop, target, time, flatten.

7. Flatten uses the clock session, not the bar session. The clock is
   authoritative.

8. Engine wiring is minimal: optional exit_engine parameter on
   ReplayEngine. Called after process_bar and before the strategy.

9. Fail-soft: if one symbol raises, the remaining symbols are still
   evaluated.

10. No journal. Stage E.

## 2026-10-02 - Lesson: bar session vs clock session
The flatten test set bar.session to FLATTENING but left the clock at
09:32 ET (TRADING). The exit engine correctly read the clock, not the
bar. Test bar and clock must be consistent.

## 2026-10-02 - Lesson: regex anchors are fragile after ruff reformats
Multiple regex-based edits failed across D9 because ruff reformatted the
file between write and edit. Safer approach: locate a function by its def
line, find the next def test_ boundary, replace the whole slice.

## 2026-10-02 - Lesson: test setup must open the account position
_setup registered a PositionPlan but did not open an account position.
broker.close_position raised BrokerError; the engine correctly caught it;
tests then expected an exit. The source was right, the setup was wrong.

## 2026-10-02 - D10 design decisions
Flatten engine:

1. Once-per-session, not per-bar. The engine tracks a per-day flag and
   calls the flat engine only when the clock first enters FLATTENING.

2. Cancel first, then close. Working entries that would otherwise fill
   during the flatten window are cancelled before positions are closed.

3. Retry with a fixed price. No price chasing. max_close_attempts bounds
   the loop. Retries continue even when an attempt made no progress,
   because the loop bound is the attempt count, not the progress flag.

4. Verify after closing, do not assume. broker.get_positions() is queried
   after the close attempts. Blueprint Section 48.

5. raise_on_failure is opt-in. Default False so the engine reports state
   without raising.

6. FlattenResult carries every symbol touched and every reason recorded.
   Stage E will persist it.

7. Session gate uses clock.session, not bar.session. Same rule as D9.

8. No datetime.now(). Day tracking uses clock.trading_day.

## 2026-10-02 - Lesson: redundant progress checks defeat retry loops
The first flatten engine broke the retry loop on "no progress". That
defeated the retry mechanism entirely, since a transient failure looks
exactly like no progress. Lesson: when a loop is bounded by a retry
count, do not add an early exit based on progress; the bound is the
contract.

## 2026-10-02 - D11 design decisions
Reconciliation engine:

1. Read-only. Reports mismatches; does not fix them. Blueprint Section 51.

2. Broker passed at call time, not construction. One engine instance can
   reconcile against any broker.

3. Symbols sorted. Deterministic iteration and output.

4. Tolerance in shares. Integer. Default 0 (exact match).

5. Only quantities compared. Average price drift, open orders, and fills
   are Stage G concerns.

6. No exceptions caught. If the broker is not connected,
   BrokerNotConnected propagates.

7. run_at from clock.now. No wall-clock dependency.

8. reasons list summarizes counts, not free-form text.

## 2026-10-02 - Stage D complete
Full event-driven replay is done. The ReplayEngine composes the feature
engine, regime classifier, news engine, opportunity scanner, AI provider,
strategy pipeline, position manager, exit engine, flatten engine, and
reconciliation engine. Blueprint Sections 33 and 67 are now structural
facts, not goals.

Next: Stage E - audit and journal. The database becomes the system's
permanent memory. Blueprint Sections 31 and 32.

## 2026-10-02 - E1 design decisions
SQLite journal:

1. SQLite, stdlib only. No new dependency. Single file. Postgres and
   DuckDB are later stages.

2. Raw sqlite3, no ORM. Explicit DDL and SQL. Easier to audit and replay.

3. Journal does not generate IDs. Callers supply deterministic IDs. This
   is what makes the D1-D11 chain reproduce identically on replay.

4. Datetimes stored as ISO 8601 with timezone. Round-trip preserves the
   offset via datetime.fromisoformat.

5. Enums stored as their .value strings.

6. JSON columns for tuple fields (risk_flags_json, checks_json, etc).
   json.dumps with sort_keys=True for determinism.

7. transaction() is the only way to write multiple rows atomically.

8. Foreign keys enforced. PRAGMA foreign_keys = ON.

9. WAL mode. Concurrent reads while writing.

10. No schema versioning in E1. Schema version constant is recorded for
    Stage F to build migrations on.

## 2026-10-02 - Lesson: ruff PYI063 and PYI034 on Pydantic context
model_post_init has a positional-only __context parameter; ruff wants
/, before the type. __enter__ returning the class should use Self rather
than the class name. Both are one-line fixes but ruff auto-fix does not
apply them.

## 2026-10-02 - E2 design decisions
Trace query:

1. market_event_id added to ai_decisions as a nullable column. Documented
   schema gap from D-stage code. E3 wiring populates it. E2 uses it
   opportunistically.

2. Helpers by decision_id, not by row id. The natural key for validation,
   risk, and portfolio is decision_id. Callers do not know row ids.

3. missing_links names the broken step. Never raises. Never guesses.

4. Read-only. No writes. No backfill.

5. Closed trades vs open trades. A trade without an exit fill is a
   legitimately open trade, not a broken chain.

6. No recursion. Linear lookups.

## 2026-10-02 - Lesson: FK constraints vs corrupt-row tests
The trades table declares a foreign key on exit_fill_id. A test that
wanted to simulate a broken exit_fill link could not insert the corrupt
row because SQLite rejected it. Fix: disable foreign_keys for that single
insert, re-enable after. The FK is correct; the test just needed a
narrower scope for its corruption.

## 2026-10-02 - Lesson: audit spine now closes the loop
trace_trade walks from a trade back to the market event that triggered
it. Blueprint Section 32 is now a structural fact, not a goal. E3 will
populate every link during replay.

## 2026-10-02 - E3 design decisions
ReplayEngine journal wiring:

1. Journal is optional. Existing tests pass unchanged when journal=None.

2. One transaction per bar. Atomic. Partial bar writes do not land.

3. Deterministic IDs generated by journal_id. Same inputs -> same IDs.
   Replay reproducibility extends to the journal.

4. Journal failure aborts the run. In-memory state after a failed bar is
   not trusted.

5. Flat engine results are not yet journaled as trades. Documented
   limitation. E4.

6. Trade creation from exit results is deferred to E4 because ExitResult
   lacks entry_fill_id and quantity.

7. experiments row is written once on the first bar when experiment_id
   is supplied. Constructor stays cheap.

8. checks_json uses model_dump on each check model, then json.dumps with
   sort_keys=True.

9. proposal provider_name and provider_version are carried on
   ProposalBundle and surfaced on ReplayStep.

10. dataset_id participates in event_id generation. A changed dataset
    produces different event_ids.

## 2026-10-02 - Lesson: propose_safely masks import errors
A missing DecisionAction import in a test provider caused propose_safely
to catch NameError and return HOLD. The engine then never placed orders,
never filled, and never wrote ai_decisions. The failing tests looked like
a journal wiring bug. Lesson: fail-safe wrappers hide bugs in test
fixtures. When a test expects a decision and sees HOLD, check the provider
imports first.

## 2026-10-02 - Lesson: transaction() and per-insert commits
Journal.insert_* methods each commit by default. Inside a transaction()
context we want one commit at the end, not one per insert. Solved by a
_suppress_commit flag that is set during transaction() and restored
afterward.

## 2026-10-02 - E4 design decisions
Trades and flat closes:

1. One fill row per order. Partial fills are summarized on the orders
   row. Deterministic IDs matter more than partial fill archaeology.

2. Deterministic fill IDs from order id alone:
   entry_fill_id = journal_id(order_id, "entry")
   exit_fill_id  = journal_id(order_id, "exit")
   trade_id      = journal_id(order_id, "trade")
   Both writer and reader can compute these without a shared clock or
   index.

3. Trades written in the same bar transaction as the exit. Atomic.

4. Fixed write order respects FKs: experiment, market_event, orders,
   entry fills, exit fills, trades, ai, validation, risk, portfolio.

5. Flat closes are trades, not system_events. E3 recorded flat runs as
   system_events because FlatClose did not exist. E4 introduces
   FlatClose and changes to trades.

6. realized_pnl comes from ExitResult and FlatClose, not recomputed.

7. entry_time comes from the plan, not from the fills table.

8. No schema change. E1's trades table already had the right columns.

## 2026-10-02 - Lesson: object-typed parameters erase mypy info
Using "object" as a type for exit_results and flat_result caused 19 mypy
errors on attribute access. Type them with the actual classes: ExitResult
and FlattenResult. The engine already imports these elsewhere; no cycle.

## 2026-10-02 - Lesson: regex replacing a def can duplicate signature
A regex that replaces from "def _write_journal" up to "def _roll_day_if_needed"
left a duplicate def line when the replacement string also ended with the
next function's def. Always include the trailing def in the lookahead
(positive lookahead (?=...)) so it is not consumed, and never include it
in the replacement text.

## 2026-10-03 - E5 design decisions
Extended journal writes:

1. Per-bar diagnostics channel. EngineContext carries a mutable
   BarDiagnostics object. The pipeline writes what it computed. The
   engine reads after the callback returns. The strategy_fn signature is
   unchanged.

2. BarDiagnostics is a plain class, not Pydantic. Per-bar scratch space.
   No validator overhead.

3. EngineContext stays frozen; the reference is fixed, the contents are
   not. Documented.

4. Journal writes are opportunistic. If a diagnostic is None, no row.

5. News rows are per record, not per snapshot. One row per news item.

6. Deterministic IDs for every new row:
   snapshot_id    = journal_id(event_id, "features")
   regime_id      = journal_id(event_id, "regime")
   opportunity_id = journal_id(event_id, "opp")
   news_id        = journal_id(event_id, "news", record.news_id)

7. No positions, health_events, system_events in E5. E6.

## 2026-10-03 - Lesson: monkey-patching methods breaks mypy
Attaching methods to a class after definition works at runtime but mypy
cannot see them. Rewrite as normal class-body methods. If a script must
modify many methods, rewrite the whole file rather than patch it.

## 2026-10-03 - Lesson: single-letter variable names collide
Using "o" for both an Opportunity and an order caused 8 mypy errors in one
function. Use full names (opp, ordr, ev, rec) inside any function that
touches more than one domain object.

## 2026-10-03 - E6 design decisions
Experiment manifest:

1. Manifest is a frozen, extra-forbid value object. Same pattern as
   every other Pydantic model in the project.

2. All version fields are caller-supplied strings. No auto-detection
   from git.

3. JSON storage with sort_keys=True. Same manifest -> same config_json.

4. Manifest's experiment_id overrides the constructor's experiment_id.

5. Compat path: if only experiment_id is given, a minimal manifest is
   synthesized with 0.0.0 versions. Existing tests pass unchanged.

6. Write-once. Second insert with same experiment_id raises
   IntegrityError. Callers use new IDs to re-run.

7. get_manifest is a convenience wrapper over get_experiment + from_json.

8. Dates as ISO strings, not date objects. Keeps DDL simple.

9. random_seed is required. Determinism is explicit.

10. Field validators reject empty strings on ID and version fields.
    model_post_init enforces start_date <= end_date, non-empty universe,
    timezone-aware created_at.

## 2026-10-03 - Stage E complete - full project review
Stages A through E are complete. What exists:

Stage A (Simulation foundation):
  - Canonical MarketEvent
  - SimulationClock with session derivation
  - HistoricalDataProvider and PointInTimeEventStream
  - Dataset validator (fail-closed)

Stage B (Simulated broker):
  - SimulatedAccount with Decimal money
  - SimulatedOrder state machine
  - Abstract Broker + SimulatedBroker
  - Fill engine (BUY + SELL, spread, slippage, commission, partial fills)

Stage C (Decision pipeline):
  - AIProposal schema
  - Deterministic validator
  - Risk engine with sizing and exposure caps
  - Portfolio engine with combined and pending exposure
  - ExecutionAuthorization and execute_authorized

Stage D (Complete replay):
  - Feature engine
  - Opportunity scanner
  - Regime classifier
  - News engine
  - AI provider abstraction and HeuristicProvider
  - Strategy pipeline
  - Position manager
  - Exit engine
  - Flatten engine
  - Reconciliation engine
  - ReplayEngine event loop

Stage E (Audit):
  - SQLite journal with 16 tables
  - 13 typed insert/get pairs
  - trace_trade walking the audit chain
  - Per-bar engine writes in a single transaction
  - ExperimentManifest persistence

Nothing in stages F-H has been built or run. The system has never seen
real historical data. It has never connected to a broker. The AI layer
has never called a real model. That is Stage F and beyond.

## 2026-10-03 - F1 Step 1 design decisions
CSV loader and writer:

1. ISO 8601 with explicit offset. Matches MarketEvent requirement.

2. Pipe-separated quality flags (comma is the CSV delimiter).

3. Required columns are the ones with no sensible default.

4. No sorting, no deduplication. HistoricalDataProvider and the dataset
   validator handle those.

5. Row number in every error. Non-negotiable for debugging.

6. write_csv exists for symmetry. One format definition.

7. Stdlib csv only. No pandas. No new dependency.

8. No IBKR import. Step 1 is offline and testable.

9. Default source is HISTORICAL.

10. Empty list on header-only file. Legitimate case.

## 2026-10-03 - Stage F decisions locked
- Vendor: IBKR. Trader Workstation with the API enabled. The IBKR API
  returns 1-minute bars through the API, capped per request and paced at
  roughly 60 requests per 10 minutes. Fine for one day, workable for a
  week. For multi-year we will need a vendor migration. Re-evaluate at F5.
- Universe: AAPL only.
- Timeframe: 1-minute.
- First date: 2026-09-08. Verified XNYS session.
- Session: regular trading hours only (useRTH=True).

Known IBKR limitations accepted for F1-F4:
- No adjusted prices. For recent dates there are no splits or dividends
  to adjust for. For older dates this would matter.
- Requires TWS or IB Gateway running locally, logged in, API enabled.
- Rate limited. Pagination for long ranges.

## 2026-10-03 - F1 Step 2 decisions and IBKR gotchas
IBKR historical downloader:

1. Script, not package code. Lives in scripts/. Not imported by app/.

2. ib_async 2.1.0. The maintained fork of ib_insync.

3. Read-only API, StartupFetchNONE. The readonly flag prevents any
   order placement. StartupFetchNONE skips the post-connect sync phase
   that was hanging on first connection.

4. Regular trading hours only (useRTH=True). No pre-market or
   after-hours bars.

5. One day, one symbol, one timeframe. Smallest possible test.

6. No retry, no pagination. Fail loudly on one attempt.

7. Timestamps normalized to America/New_York before constructing
   MarketEvent.

## 2026-10-03 - macOS TWS binds IPv6 only
TWS on macOS listens on IPv6 (lsof shows IPv6 *:7497). It does not
accept IPv4 connections on 127.0.0.1. Use --host localhost, which
resolves to ::1 (IPv6 loopback). This is a macOS-specific behavior;
Linux TWS binds both. Documented for future IBKR work.

## 2026-10-03 - First real data validated clean
data/raw/aapl-2026-09-08-1m.csv: 390 bars, 09:30 to 15:59 ET, prices in
the 315-320 range. load_csv parses it. validate_dataset reports
is_valid=True with zero issues. This is the first time real market data
has passed through the Stage A tooling. The tools work.

## 2026-10-03 - What F2 is actually for
Every previous step tested components in isolation. F2 is the first
integration test with real inputs. Expect at least one of:
- The scanner returns zero opportunities all day.
- The AI returns HOLD on every bar.
- The pipeline produces a proposal but the risk engine rejects it.
- A fill never happens because the next bar has no volume.
- The journal writes something we did not expect.

None of these are bugs. They are data. F2's job is to reveal them.

## 2026-10-04 - F2 design findings
First one-day replay on real market data (AAPL 2026-09-08):

Finding 1 - Pipeline integrity
The full pipeline ran end to end on 390 real bars with no crashes,
no silent failures, and a complete journal. Every layer recorded what
it did. trace_trade-style queries resolve even for non-filled
proposals. The system is structurally sound.

Finding 2 - Scanner behavior
5 opportunities out of 331 bars (1.5%). RVOL at opportunity time was
3.04 to 7.95. RSI at opportunity time was 52 to 69. The RVOL > 3
filter is doing real work; on a normal AAPL day it fires on a handful
of minutes, not on all of them.

Finding 3 - Trend gate dominance
3 of 4 HOLD decisions fired because sma_fast < sma_slow (trend DOWN).
RSI at those moments was 55 to 69, i.e. momentum-positive. The
heuristic requires trend UP; the SMA relationship disagreed. This is
not a bug. It is a design characteristic of the HeuristicProvider on
this day. Whether it generalizes is an F3 question.

Finding 4 - Structural coupling (recorded, not fixed)
The heuristic sets confidence = clamp(opportunity_score, 0, 1). The
validator requires confidence >= min_confidence (default 0.70). The
heuristic emits BUY when opportunity_score >= min_opportunity_score
(default 0.50). Therefore any BUY with score in [0.50, 0.70) is
structurally doomed: proposed, then rejected. On 2026-09-08, the one
BUY had confidence 0.6287, right in the dead zone.

This coupling is a design flaw. It is not a bug in the sense that the
system behaves as coded. But it wastes AI calls, logs rejections that
could not have succeeded, and hides the intent of the heuristic.

Three options for later:
A. Accept it. Document that BUYs below 0.70 are always rejected.
B. Heuristic self-throttle: refuse to emit BUY when its own
   confidence would be below the validator threshold. Requires
   matching config values.
C. Decouple: heuristic derives confidence from AI-relevant signals
   (regime quality, feature strength, trend strength) rather than
   reusing opportunity_score.

Do not decide on n=1. Collect F3 data first.

## 2026-10-04 - Audit gap fix
E3 wrote ai_decisions only when a ProposalBundle existed. When the
pipeline returned None because the heuristic returned HOLD, the AI
decision was invisible. This hid 4 of 5 AI calls on 2026-09-08.

Fix: BarDiagnostics gained ai_response; the pipeline writes the AI
response into diagnostics; the engine writes ai_decisions from
diagnostics when present, even when no step exists. The engine also
writes ai_decisions from step when diagnostics is absent for
backward compatibility.

After the fix, Run 2 shows 5 ai_decisions (4 HOLD, 1 BUY) matching
the 5 opportunities. Blueprint Section 32 (nothing important is a
black box) is restored.

## 2026-10-04 - Process note: regex on already-formatted code
Two consecutive attempts to modify BarDiagnostics failed because ruff
had reformatted __slots__ into a multiline tuple and then into a
single line, and my regexes assumed a specific shape. The fix was to
abandon regex and remove __slots__ entirely. Lesson: when an edit
targets a small, well-defined region and automated matching keeps
failing, edit the region directly (via a full-file rewrite or by
removing the construct entirely) rather than trying harder regexes.
