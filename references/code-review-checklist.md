# Code review checklist

The reviewer gets the diff, the spec or issue, and the tests. Give them the code, not your conclusion.

## 1. Spec compliance
- [ ] Does it do what was asked, all of it, and nothing extra?
- [ ] Every requirement maps to a test.
- [ ] Deviations are recorded as rulings with who decided, not slipped in.

## 2. Correctness
- [ ] Null, empty, zero, negative and boundary values.
- [ ] Off-by-one, rounding and currency precision (never floats for money), dates and time zones.
- [ ] Every branch of a conditional is reachable and tested.
- [ ] Error paths: what happens when a call fails halfway? Is a partial write left behind?
- [ ] Idempotency: a double click, a retried job or a duplicate webhook does not create twice.
- [ ] Concurrency: two users or jobs on the same row; read-modify-write without a lock; check-then-act races.
- [ ] Transactions: the commit and rollback boundaries are explicit; no rollback to a savepoint after the
      DB has already aborted the transaction; no commit inside a loop that leaves a half-done chain.
- [ ] Cancel, amend, return and undo flows reverse everything the forward flow did.
- [ ] Feature flag Off is fully inert: no queries, no side effects, no UI.

## 3. Cross-cutting effects (the whole-change review)
- [ ] Hook, event and signal order across models; overrides of framework methods call `super`.
- [ ] Shared state: globals, request-local caches, module caches invalidated on change.
- [ ] Migrations are reversible or have revert steps; they are safe on large tables (locks, batching).
- [ ] Background jobs: retries, timeouts, and marking done only after the work is done.
- [ ] Revert order: what breaks if only this PR is reverted?

## 4. Quality
- [ ] It reads like the surrounding code: naming, idiom, comment density.
- [ ] No dead code, debug prints, commented-out blocks or TODOs without a ticket.
- [ ] Duplicated logic reuses the existing helper.
- [ ] Settings are not hard-coded where the project uses configuration.
- [ ] Tests assert behaviour, not implementation, and fail for the right reason.
- [ ] User-facing text is clear and actionable.

## 5. Verdict format
For each finding: file:line, severity (P1 blocks merge / P2 fix before release / P3 nit), what is wrong,
a concrete failure scenario, and a suggested fix. Findings without a failure scenario are opinions; mark
them as such.
