# Debug rule: no fix without a root cause

Applies to every red at every gate. Symptom fixes are failures, and guessing is slower than the method.

## Phase 1: Investigate (before touching code)
1. **Read the whole error**: the full stack trace, error code, file and line, and the first error rather
   than the last. DB codes matter: 1205 lock wait, 1213 deadlock, 1020 record changed, 1305 savepoint
   missing, 40001 serialization failure.
2. **Reproduce it** on a local or disposable copy, with the exact steps written down. If it won't
   reproduce, gather more data rather than guess.
3. **Check what changed**: `git log`/`git diff <base SHA>`, dependency bumps, config and env vars, feature
   flags, migrations, caches still holding old code, data that differs between environments.
4. **Instrument the boundaries** where a flow spans layers (UI → API → service → framework hook → DB →
   external API). Log what goes in and comes out at each boundary, run once, and see **where** it breaks
   before asking why.
5. **Trace backwards** from the bad value: who passed it, and who passed it to them? Fix where the value
   is born, not where it explodes.

## Phase 2: Compare
- Find the nearest flow that works: the same path with the flag Off, the base commit, another record or
  tenant, another environment, the stock framework behaviour.
- List **every** difference, however small. Don't decide something "can't matter" without testing it.
- If you are using a reference pattern, read the whole reference before copying it.

## Phase 3: One hypothesis at a time
- Write it down: "X is the root cause because Y."
- Test it with the **smallest** change, one variable at a time.
- If it is wrong, form a new hypothesis. Never stack a second fix on top of the first.
- If you don't understand something, say so and research it or ask. Don't pretend.

## Phase 4: Fix
1. Write a failing test that reproduces the bug (RED), using the simplest reproduction.
2. Make one fix at the root cause. No "while I'm here" changes.
3. GREEN, then run the whole suite, then re-run the gate that found the bug.
4. If the fix doesn't work and you have tried fewer than 3, go back to Phase 1 with what you learned.
   **At 3, stop**: each fix uncovering a new problem elsewhere means the design is wrong. Take it to the
   user before a fourth attempt.

## Flaky tests
Flaky means not yet understood. Look for order dependence, shared fixtures, test pollution (bisect the
order to find the polluter), real time and time zones, randomness without a seed, network, lock timing,
and caches. Never re-run until green.

## When the cause isn't in our code
The cause can be the framework, the environment, timing or a third party. Record what was checked and
what ruled our code out. Then add handling: a clear message, a bounded retry, a timeout, or logging that
will catch it next time. Most "no root cause" findings are investigations that stopped too early.

## Red flags: stop and go back to Phase 1
- "Quick fix for now, investigate later"
- "Let me just try X and see"
- Several changes at once, then running the tests
- "I'll verify it by hand, no test needed"
- Proposing fixes before reproducing the bug
- "One more attempt" after two have failed

## What goes in the gate report for each bug
| Symptom | Where found (gate) | Root cause | Evidence (log line / query / diff) | Fix (commit) | Guarding test |
