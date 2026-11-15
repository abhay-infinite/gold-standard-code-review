## What and why
<One paragraph: the behaviour change and the requirement or issue it answers.>

## Gold standard gates
| Gate | Result | Evidence |
|---|---|---|
| TDD | RED → GREEN | `<test name>`: failed on <sha>, passes on <sha> |
| Code review | | |
| Security | | |
| Performance | | Off vs On: … |
| Integration | | full suite vs baseline: … |
| Error-log / past errors | | |
| UX / errors | | screenshots |

## Edge cases covered
- [ ] Flag Off is inert  - [ ] Empty/zero  - [ ] Cancel/return  - [ ] Retry/idempotent
- [ ] Two users at once  - [ ] Permissions per role  - [ ] Unmigrated site

## Bugs found and root-caused
| Symptom | Root cause | Guarding test |
|---|---|---|

## Revert
- Reverting this PR leaves behind: <data / settings / none>
- Revert SQL or steps: <none / …>

## Deploy notes
- Runbook steps beyond the standard deploy: <none / …>
- Feature flag default: <Off / On, and why>
