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
