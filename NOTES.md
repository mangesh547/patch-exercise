# NOTES

## Summary of changes
- **Search SQL (highest value):** `AND`/`OR` precedence bug. Archived tasks leaked into results and the status filter was ignored for title matches. Parenthesised the title/description match; same fix in `db/queries/search_tasks.sql` and both queries in the Oracle package. Added `id DESC` tie-breaker so pagination order is stable.
- **Controller:** removed a `Thread.sleep` that made short or empty searches up to 1s slower; invalid `status` now returns 400 instead of 500; `page`/`pageSize` are clamped (no negative `subList` index, max page size 100); `System.out` replaced with SLF4J.
- **Frontend hook:** loading stayed `true` forever after an error; errors never cleared; out-of-order responses could overwrite newer results (fixed with `AbortController`).
- **UX:** 300ms search debounce, page resets to 1 on any new search/filter, removed a debug `console.log`.

## Not changed
- Pagination is still done in memory (fetch all, then `subList`). Moving it to the DB (`Pageable` + count query) is right but a bigger diff than a patch exercise wants.
- LIKE wildcards (`%`, `_`) typed by users are not escaped.
- `@CrossOrigin` is hardcoded to localhost:5173; should be config.

## Biggest remaining risk
Unbounded in-memory result sets plus no automated tests. Search is the core feature and nothing guards it against regressions like the precedence bug.

## Tools / AI used
Used Claude (Claude Code) to review the codebase and draft the fixes. I checked the SQL fix by loading `data.sql` into SQLite and comparing old vs new queries (old query leaked 2 archived rows for "api"; new returns 0). I reviewed and understood each change; handwritten notes are in `handwritten/`.
