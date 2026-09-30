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
