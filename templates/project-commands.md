# <Project>: gold standard commands

> Copy to `<repo>/.gold-standard/commands.md`. Fill it in during Step 0 by reading CI config and build
> files, and prove each command once. Add traps as they are found (Step 10).

## Places
- Repo / default branch / PR base branch:
- Local dev environment (how to start it):
- Disposable copy (how to create one for stress, perf and anything that commits):
- Staging (host, how to run commands there, read-only or not):
- Production (read-only unless explicitly authorised):

## Gates
| Gate | Command | Approx. time | Notes |
|---|---|---|---|
| Full test suite | | | |
| Single test / file | | | |
| Lint | | | |
| Type check | | | |
| Build | | | |
| Migrations check | | | |
| Dependency audit | | | |
| Perf / query count | | | |
| Concurrency / stress | | | |
| Browser end to end | | | |
| Error log query | | | |

## Caches to clear when switching code
-

## Deploy
```bash
# backup → clean tree → pull --ff-only → migrate → build → runbook steps → clear caches → restart → smoke
```

## Post-deploy checks
- Deployed SHA:
- Error log since deploy:
- Smoke flows (role user, not admin):

## Traps
- <date>: <what bit us, and how to avoid it>
