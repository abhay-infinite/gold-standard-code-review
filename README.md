# Gold Standard Review: a Claude Code skill

Quality gates for any project, in any stack:
**TDD → systematic debugging → code review → security → performance → integration → error-log gate →
clear errors and UX → safe delivery with post-deploy checks**, plus a past-errors list that grows each run.

## Install
```bash
# for every project on this machine
git clone https://github.com/abhay-infinite/gold-standard-code-review.git ~/.claude/skills/gold-standard-review

# or for one repo only (it can be committed and shared with the team)
git clone https://github.com/abhay-infinite/gold-standard-code-review.git <repo>/.claude/skills/gold-standard-review
```
Restart Claude Code (or start a new session) so the skill is picked up. To update later: `git -C ~/.claude/skills/gold-standard-review pull`.

## Use
In Claude Code, say **"run gold standard review"** (or "gold standard", "run the gates", "make it go-live
ready"), and name the scope: a PR number, a branch, or "release = staging vs last tag".

On the first run in a project, Claude creates `.gold-standard/` in the repo:
- `commands.md`: the project's real test, lint, build, deploy and log commands
- `past-errors.md`: every failure mode the project has had, each with the test that guards it
- `runs/`: one gate report per run

Commit that folder, so the standard grows with the project and the whole team uses the same one.

## Files
| File | What it is |
|---|---|
| `SKILL.md` | the gates and rules Claude follows |
| `references/debugging.md` | the Debug rule (no fix without a root cause) |
| `references/code-review-checklist.md` | spec, correctness, cross-cutting, quality |
| `references/security-checklist.md` | authorization, forged fields, injection, secrets, CVEs, licences |
| `references/performance-checklist.md` | Off/On overhead, budgets, N+1, locks, retries, load |
| `references/known-failure-modes.md` | 26 failure modes seen across projects |
| `references/stack-notes.md` | where the commands live for Frappe, Django, Node, React, Go, Java |
| `templates/*.md` | project commands, past errors, gate report, PR body |

## Licence
MIT. See [LICENSE](LICENSE). Use it, fork it, adapt it to your team.
