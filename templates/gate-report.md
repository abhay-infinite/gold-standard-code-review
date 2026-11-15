# Gold standard run: <scope>, <date>

- **Change set:** <branches / PRs / release range>
- **Base SHA:** <sha>    **Head SHA:** <sha>
- **Environments used:** <local / disposable copy / staging>

## Gate table
| # | Gate | Status | Evidence (command + result) | Notes |
|---|---|---|---|---|
| 1 | Baseline | | e.g. `pytest` → 412 passed, 3 failed (pre-existing: …) | |
| 2 | TDD evidence (RED → GREEN) | | | |
| 3 | Code review (per PR + whole change) | | | |
| 4 | Security review | | | |
| 5 | Performance (6 checks) | | | |
| 6 | Integration vs baseline (Off and On) | | | |
| 7 | Error-log + past-errors gate | | | |
| 8 | Clear errors + UX walk | | | |
| 9 | Delivery + post-deploy verification | | | |
| 10 | Standard updated | | | |

Status: **Pass** / **Fail** / **N/A (reason)**. Never blank.

## Bugs debugged
| Symptom | Found at gate | Root cause | Evidence | Fix (commit) | Guarding test |
|---|---|---|---|---|---|

## Rulings (deviations from spec)
| What | Decision | Who | Date |
|---|---|---|---|

## Open P1s
-

## Revert notes
- What a revert of each PR leaves behind (data, settings, migrations), and any revert SQL.

## What changed in the standard
- New past-errors entries:
- New commands or traps:
