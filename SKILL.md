---
name: gold-standard-review
description: Runs gold-standard quality gates on any project, in any stack (Frappe/ERPNext, Django, Node/TypeScript, React, Go, Java and others), for a change, a set of PRs or a release. The gates are TDD evidence, systematic root-cause debugging of every red, independent code review, security review, performance and concurrency checks, integration regression against a baseline, a production error-log gate so no past error comes back, clear error messages and UX, safe delivery with post-deploy verification, and a per-project past-errors list that grows each run. Use when the user says "gold standard", "golden standard", "run the gates", "gold standard review", "make it go-live ready", "make sure everything is working", or asks for a full quality review before merge or release. If the project has its own project-specific gold-standard skill, use that instead.
---

# Gold Standard Review

The bar:
**TDD first → code review → security review → performance testing → integration testing**. Edge cases are
always covered, errors are handled clearly, and no past error is allowed back. A third party may review
the work, so every gate leaves evidence that stands on its own: in the PR description, or in the gate
report for a release.

This skill works on any project. What is specific to a project (commands, sites, environments, its own
past errors) lives **inside that project**, in `.gold-standard/`, not in this skill:

| File in the project | Purpose | Starts from |
|---|---|---|
| `.gold-standard/commands.md` | how to test, lint, run, profile, deploy and read logs here | `templates/project-commands.md` |
| `.gold-standard/past-errors.md` | every failure mode this project has had, and the test that guards it | `templates/past-errors.md` |
| `.gold-standard/runs/<date>-<scope>.md` | the gate report for each run | `templates/gate-report.md` |

Reference files in this skill (read the one a step points to, when you reach that step):
- `references/debugging.md`: the Debug rule in full
- `references/code-review-checklist.md`
- `references/security-checklist.md`
- `references/performance-checklist.md`
- `references/known-failure-modes.md`: failure modes that recur across projects, as a starting list
- `references/stack-notes.md`: commands and traps per stack

---

## Rules at every step

**1. Debug rule: no fix without a root cause.** Any red at any gate is debugged systematically before any
code changes. That covers a failing or flaky test, a number over budget, a deadlock, an error-log
signature, a console error, a screen that looks wrong and a failed smoke check. The cycle is investigate →
compare with something that works → one written hypothesis, tested minimally → failing test → one fix at
the source → re-run the gate. **Three failed fixes = stop** and take the design question to the user.
Full method: `references/debugging.md`. If the `superpowers:systematic-debugging` skill is installed, use
it as well.

**2. Evidence rule: nothing is "working" without fresh output.** Before any gate is marked Pass, and before
saying anything is fixed, done or working, run the command that proves it in this session, read the full
output (exit code, fail count) and quote it. A sub-agent's "all green", an earlier run, or "should pass"
is not evidence. Re-run it and check the diff yourself.

**3. Never on production.** Commands that write, stress or migrate run only on a local or disposable copy.
Production is read-only unless the user explicitly authorises a write.

**4. One change, one PR, one revert.** Every PR can be reverted on its own; say what runtime data a revert
leaves behind.

---

## Step 0: Pin the scope and learn the project
- Name the change set: branch(es), PR numbers, or "release = <env> vs <tag/date>". Record the base SHA.
- If `.gold-standard/commands.md` is missing, build it now from the project's own files (CI config,
  `Makefile`, `package.json`, `pyproject.toml`/`setup.cfg`, `go.mod`, `pom.xml`, `docker-compose*`, README,
  CLAUDE.md), using `references/stack-notes.md` for the stack. Run each command once to prove it works
  before relying on it. Ask the user only for what the repo cannot tell you (staging host, log access).
- If `.gold-standard/past-errors.md` is missing, copy the template and seed it from the project's history:
  `git log --grep='fix'`, reverted commits, incident notes, the issue tracker.
- Start the gate report (`templates/gate-report.md`): one row per gate with status, evidence and notes.

## Step 1: Baseline
- Run the full test suite, lint and type checks on the **base** code and save the log. Tests that already
  fail there set the bar; they aren't blamed on the change. Each one still gets a root cause written down.
- Clear every cache that can carry old code: build caches, compiled assets, framework/app caches, hooks
  cached in redis, `__pycache__`, `node_modules/.cache`. Restore afterwards anything that the run changed.

## Step 2: TDD evidence
- Every behaviour change has a test that was run and seen failing (RED) before the code, then passing
  (GREEN). Check commit order or re-derive it: stash the fix, run, see red. A test added without a RED is
  flagged in the report, not hidden.
- Edge cases are listed and tested: feature flag Off/On, empty/zero/null, boundary values, partial
  success, cancel/undo/return, repeat/retry/idempotency, two users at once, an unmigrated or old schema,
  permissions per role, time zones and dates, large inputs, unicode.

## Step 3: Code review
- An independent reviewer for each PR or task, checking spec compliance (does it do what was asked, no
  more?) and quality. Then a fix loop with scoped re-reviews. Checklist: `references/code-review-checklist.md`.
- One review of the whole change, on the most capable model, for effects that cross files: hook/event
  order, transactions and rollback, nested calls, elevated permissions, shared state, migrations and
  revert order.
- Record every deviation from the spec as a ruling, with who decided it.

## Step 4: Security review
Use `references/security-checklist.md`. At minimum, cover these:
- **Authorization** on every entry point (API, RPC, form save, bulk edit, background job, webhook). Try a
  forged field on every write path, and decide from the **stored** record, not the request payload.
- **Elevated code** (admin/sudo/ignore-permissions blocks) never writes user-supplied values beyond its
  scope, and restores the user.
- **Injection:** SQL, template (autoescape on), shell, path, SSRF, deserialisation.
- **Data exposure:** errors, logs, exports and reports respect row-level permissions; no secrets in code,
  logs or the client bundle.
- **Dependencies:** known CVEs (`npm audit`, `pip-audit`, `govulncheck`, etc.) and licence boundaries
  (for example GPL code vs proprietary custom apps).

## Step 5: Performance (always the full set)
Use `references/performance-checklist.md`. Run only on a disposable copy.
1. Overhead with the feature flag Off vs On on the high-volume write paths.
2. Full-flow timings against stated budgets. If there are no budgets, record the numbers and propose them.
3. Query counts / N+1 per request and per job.
4. Concurrency: 2, 5 and 10 parallel flows on the same hot rows (counters, naming series, stock and
   balance rows). Look for lock waits, deadlocks, duplicate or skipped IDs, and half-done chains.
5. Retry and idempotency paths re-run with the change On.
6. Load and stress smoke, and API latency budgets, Off vs On.

## Step 6: Integration
- Run the full suite on the change and compare it line by line with the Step 1 baseline: nothing may get
  worse. Every new red goes through the Debug rule.
- Run the same suites with the feature switched **On**.
- Run the build, migrations on a copy of real data, lint, type checks and static/integrity checks.
- Exercise external integrations (payment, tax, marketplace, email) against their sandbox or mock, plus
  their failure modes: timeout, 4xx/5xx, duplicate callback.

## Step 7: Error-log gate (no past error comes back)
- Pull the application error log, failed background jobs and monitoring (Sentry or similar) from the
  shared environment since the last run. Sort each signature into one of: our bug / framework limitation
  / raw error that must become a clear message / config noise / already fixed. Every "our bug" goes
  through the Debug rule. Every "already fixed" needs the PR and the test that guards it.
- Walk `.gold-standard/past-errors.md`, plus `references/known-failure-modes.md` where it applies. Each
  entry must have a regression test, or a recorded check that still passes. **Any recurrence is a P1.**
- Browser end-to-end runs record JS console errors and failed network calls on every screen visited.

## Step 8: Clear errors and UX
- No user path ends in a stack trace, a blank screen or a silent failure. Every refusal says what is wrong,
  what to fix and where.
- Walk the changed flows on screen as a real role user (not admin), keeping screenshots as evidence.
  Anything confusing is a finding. Check phone width if users are on phones.

## Step 9: Safe delivery
- One PR per change. The PR body (`templates/pr-body.md`) carries the gate table, the test evidence, what a
  revert leaves behind, and any revert SQL or data steps.
- Deploy the way `.gold-standard/commands.md` says: backup → clean tree → fast-forward pull → migrate →
  build → runbook steps → clear caches → restart → smoke check. Check for other deploys in flight first.
- **Post-deploy verification, before saying it is live:**
  1. The deployed SHA equals the merged SHA, and migrations finished clean.
  2. The error log and failed jobs since deploy time show zero new signatures from the changed code. Any
     new one goes through the Debug rule, and the fix (or revert) comes first.
  3. The changed flows are walked in the browser as a real role user: screens load, the main action
     succeeds, the console is clean, screenshots are kept.
  4. Feature flags are in the state the PR says.
  If any check fails, fix or revert, and say which in writing.
- After deploy, update the docs (PRD, user guide or training, changelog), never before the feature is live.

## Step 10: Keep the standard growing
- Append every new failure mode from this run to `.gold-standard/past-errors.md`: signature, root cause,
  the test that now guards it. If it would bite any project, append it to
  `references/known-failure-modes.md` in this skill too.
- Append new commands and traps to `.gold-standard/commands.md`.
- Save the gate report to `.gold-standard/runs/`.
- Report to the user: the gate table (pass / fail / evidence), every bug debugged (symptom → root cause →
  fix → guarding test), open P1s, and what changed in the standard. Use only results from commands run in
  this session.

---

## Scaling the run
- **One small PR:** every gate still runs, but a gate can be short. Say what was checked and why that was
  enough ("no new queries, so perf = query-count diff only"). Skipping silently is never allowed.
- **Release:** the full set, with sub-agents in parallel for review, security and perf when available.
  Every result is re-checked under the Evidence rule.
