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
