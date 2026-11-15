# Stack notes: where the commands usually are

In Step 0, find the real commands from the repo (CI config first, since it is what actually runs), prove
each one once, and write them to `.gold-standard/commands.md`.

## Frappe / ERPNext
- Tests: `bench --site <site> run-tests --app <app> [--module …]`; custom integration scripts are often
  under `<app>/tests/`.
- Before running tests, migrate and clear the cache: `bench --site <site> migrate && bench --site <site>
  clear-cache`. Hooks are cached per site in redis.
- Disposable site: restore a backup into `<name>.localhost`. Never stress or commit on the working site.
- Error log: `select method, error, creation from \`tabError Log\` where creation > …`, plus `tabScheduled
  Job Log` where status='Failed'.
- Security hotspots: `@frappe.whitelist()` methods (`methods=["POST"]`, role checks),
  `ignore_permissions`, `frappe.db.sql` with f-strings, `frappe.client.*` generic writes, Jinja print
  formats (no autoescape), permlevels.
- Locks: `tabSeries` naming rows, `tabBin` stock rows. Check MariaDB `innodb_snapshot_isolation`.
- Deploy: backup → `git pull --ff-only` → `bench migrate` → `bench build --app <app>` → clear-cache →
  `supervisorctl restart all`.
- Licence: keep customisation in a standalone app (the framework is GPLv3).

## Django
- Tests: `pytest` / `python manage.py test`; `assertNumQueries` for N+1; `--reuse-db` carefully.
- Checks: `python manage.py check --deploy`, `makemigrations --check --dry-run`, `ruff`/`flake8`, `mypy`,
  `pip-audit`, `bandit`.
- Security hotspots: `@csrf_exempt`, `.extra()`/`.raw()`, `mark_safe`, `|safe`, missing object-level
  permission in viewsets.
- Locks: `select_for_update`, `F()` expressions vs read-modify-write; `transaction.atomic` boundaries.

## Node / TypeScript (Express, Nest, Next)
- Tests: `npm test` / `vitest` / `jest`; types: `tsc --noEmit`; lint: `eslint`; `npm audit --omit=dev`.
- Security hotspots: raw SQL in Knex/Prisma `$queryRawUnsafe`, `dangerouslySetInnerHTML`, missing auth
  middleware on a route, secrets in `NEXT_PUBLIC_*`.
- Perf: unawaited promises, sequential awaits in loops, bundle size (`next build` output).

## React / front end
- Tests: component tests + Playwright/agent-browser end to end; record console errors and failed network
  calls.
- Checks: accessibility (axe), phone width, dark mode if supported, loading, empty and error states.

## Go
- `go test ./... -race`, `go vet`, `staticcheck`, `govulncheck`; `-race` covers the concurrency gate
  in part.

## Java / Spring
- `mvn verify` / `gradle check`; `@Transactional` boundaries; `dependency-check`; Spring Security config
  per endpoint.

## Any stack: logs and monitoring
- App error table or log files, the job queue's failed jobs, Sentry/Datadog/CloudWatch. Query since the
  last run's timestamp and group by signature.
