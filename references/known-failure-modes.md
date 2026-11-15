# Known failure modes (across projects)

These are seed entries for any project's `.gold-standard/past-errors.md`. Each one has bitten a real
project. In Step 7, check the ones that apply to the stack. Append new entries that would bite any project.

| # | Signature | Root cause | Guard |
|---|---|---|---|
| 1 | Deadlock / lock wait / "record has changed" on busy saves | a shared counter or naming-series row, plus nested inserts in one request | 2/5/10 parallel-flow test; retry the whole unit at the entry point |
| 2 | "Savepoint does not exist" after a lock error | the DB already rolled the whole transaction back (MySQL/MariaDB do this on 1213, and on 1020 with snapshot isolation); the code rolled back to a savepoint | never roll back to a savepoint after a DB abort; re-raise to an outer retry |
| 3 | A per-item savepoint "isolation" doesn't survive a lock clash | same as #2: the item-level catch hides a dead transaction | re-raise lock clashes from per-item handlers |
| 4 | Background jobs fail on "record changed" while web requests don't | the framework retries lock errors in requests but not in jobs | jobs that write shared rows retry or serialise |
| 5 | ImportError / "No module named …" after testing a branch | hooks or config cached per site or app (redis, compiled cache) from another checkout | clear caches after switching code; restore afterwards |
| 6 | Every save fails on a site not yet migrated | code reads a new settings table or column without checking that it exists | treat a missing table as "feature Off"; test with the table removed |
| 7 | A user flips a "skip the check" flag | the flag is client-writable | set by the server only; dropped on user edit; forged-field test |
| 8 | Permission scope bypassed by a crafted request | the guard read the incoming payload, not the stored row | decide from the stored record; try every write path with a forged field |
| 9 | Automation did something the user couldn't | an elevated block did the final submit | act only if the triggering user may |
| 10 | Typed text rendered as HTML in a print, PDF or email | template engine without autoescape | autoescape on; test with `<b>` in the input |
| 11 | Raw stack trace pop-up for the user | validation raised as an unhandled exception, or missing master data | the error-log gate + clear-message review |
| 12 | A side-effect hook blocked the main save | lookup or automation ran outside its guard | hooks wrap and log, never raise (except DB aborts, which must propagate) |
| 13 | Scheduled job "succeeds" but nothing was sent | marked done before sending; wrong signature never exercised | job tests that really render and send; mark done after |
| 14 | Payment taken, then the order couldn't complete | validation ran after the external charge | validate fully before any irreversible external call |
| 15 | e-Invoice or third-party ID issued but not saved | cancel or edit allowed while the external call was pending | block state changes while an external call is pending |
| 16 | Every row of a report or mail shown in the first row's currency or unit | formatter cached per (doctype, name) = (None, None) | format each row from its own column options; a multi-currency test |
| 17 | The same lookup run hundreds of times per request | a tree or ancestor walk with one query per level, per line | per-request cache, invalidated on change; a query-count test |
| 18 | Retry added but not measured | the perf harness called the inner method, not the user's entry point | perf and retry tests call the real entry point |
| 19 | A print or default "sticks" to the next document | the UI view is a singleton that keeps its state | clear state on navigation; test in the SPA without reloading |
| 20 | Staging emails real people during tests | outbound mail not restricted on staging | allow-list mode checked before every run |
| 21 | A dev server serves the wrong site | default-site fallback ignores the Host header | always pass the site explicitly |
| 22 | An asset build from a branch ships unmerged JS to every site | assets are shared bench- or monorepo-wide | build on an isolated workspace, or inject the file in the browser only |
| 23 | An empty datetime read as a truthy "0001-01-01" | framework default for empty singles or settings | read raw and treat empty explicitly |
| 24 | A batch rebuild rolls everything back for one bad item | all-or-nothing transaction around a loop | skip and report conflicting items; one rebuild at a time |
| 25 | SwiftUI sidebar rows look fine but clicks do nothing (only ⌘-shortcuts reach them) | on macOS 26, rows placed straight into `List(selection:)` (not inside `ForEach`) ignore mouse selection even with a matching `.tag` | build every selectable row from `ForEach`; a test that sends a real click to each row and reads the AX selected row |
| 26 | A screen spins forever or shows its empty state when its data call fails | load wrapped in `try?`, the error dropped | keep the error in state and show it with a Try again; test with a backend that exits 1 |
