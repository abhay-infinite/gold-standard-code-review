# Performance checklist (always the full set)

Run on a disposable copy with realistic data volume, never production. Record numbers On vs Off, and
before vs after.

## 1. Overhead on hot write paths
- Identify the highest-volume writes (orders, invoices, stock moves, POS bills, imports, webhooks).
- Time N saves with the feature flag Off and On; report p50 and p95 and the delta. Off must equal the base.

## 2. Full-flow timings against budgets
- Time end to end what the user waits for (click → result), including chained documents and jobs.
- If no budgets exist, record the numbers and propose budgets (for example p95 < 1 s for interactive
  saves, < 300 ms for reads).

## 3. Query counts and N+1
- Count queries per request and per job (the framework's query log, Django `assertNumQueries`, ORM
  logging, `EXPLAIN`).
- A query inside a loop over rows or lines is a finding. So is the same statement repeated: cache it per
  request.
- New queries on large tables have an index; check `EXPLAIN`.

## 4. Concurrency and locks
- Run 2, 5 and 10 parallel flows on the **same** hot rows: counters and naming series, balances and stock
  rows, parent records with many children.
- Check for lock waits, deadlocks (MySQL 1205/1213/1020, Postgres 40P01/40001), duplicate or skipped
  IDs, lost updates, and half-done chains (A created, B not).
- Verify that retry happens at the **entry point the user hits** and retries the whole unit, and that a
  retry after a DB abort never rolls back to a savepoint (the transaction is already gone).
- Check DB isolation settings per environment (for example MariaDB 11.8
  `innodb_snapshot_isolation`). A setting that differs between staging and production changes this result.

## 5. Retry and idempotency
- Re-run retry suites with the change On.
- Inject a failure mid-flow: does a retry finish cleanly without duplicates?

## 6. Load, stress and API budgets
- Smoke stress (a burst of concurrent users) and soak (sustained load) where the change is hot.
- API latency budgets per endpoint, On vs Off.
- Front end: bundle size delta, the time the changed screen takes to become interactive, no extra request
  waterfalls.

## Report format
| Check | Off | On | Budget | Pass/Fail | Notes |
