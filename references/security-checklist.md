# Security review checklist

## Authorization (the most common real finding)
- [ ] List every entry point the change touches: HTTP routes, RPC/whitelisted methods, form saves, generic
      CRUD endpoints (REST PUT/POST, bulk edit, import), background jobs, webhooks, CLI commands.
- [ ] Each one checks authentication, role, and object-level permission (can **this** user touch **this**
      record?).
- [ ] State-changing methods accept POST (or the right verb) only, with CSRF protection where the
      framework expects it.
- [ ] **Forged-field test:** on every write path, send a field the UI never sends (owner, status, amount,
      parent ID, a control-skipping flag). Decisions must come from the stored record, not the payload.
- [ ] Flags that bypass a control are set by the server only, and are dropped when a user later edits the
      record.
- [ ] Elevated blocks (admin, sudo, `ignore_permissions`, service accounts) are as small as possible,
      write no user-supplied value beyond their scope, and restore the original user even on error.
- [ ] An automated action never achieves more than the triggering user could (for example, submitting a
      document they can't).

## Field- and row-level data control
- [ ] Money, tax, pricing and settings fields are protected by field-level permissions where the framework
      supports them.
- [ ] Reports, exports, search, autocomplete and dashboards respect row-level restrictions.
- [ ] Error messages and logs don't leak data to users who can't read the source.

## Injection
- [ ] SQL: parameterised queries only; no string-formatted SQL with user input; dynamic identifiers
      (column, table, order-by) are allow-listed.
- [ ] Templates: autoescape is on (watch for print, PDF and email templates that turn it off); `|safe`
      and raw HTML are justified.
- [ ] Shell, path traversal, file upload (type, size, location), SSRF on URL fetches, unsafe
      deserialisation (pickle, YAML load), regex DoS.

## Secrets and config
- [ ] No secrets in code, tests, fixtures, logs, the client bundle or the PR description.
- [ ] Debug modes, test endpoints and mock gateways are off in production config.
- [ ] Outbound email or SMS is restricted on staging.

## Data changes
- [ ] Patches and migrations stay inside their WHERE clause; dry-run counts are recorded before and after.
- [ ] Deletes are soft, or backed up; cascades are checked.

## Dependencies and licences
- [ ] `npm audit` / `pip-audit` / `govulncheck` / `mvn dependency-check`: no new high or critical CVEs.
- [ ] Licence boundary respected (for example GPL framework code vs a proprietary custom app; no copying
      core code into a closed app).

## Output
For each finding: entry point, attack (the exact request), impact, fix, and the test that now guards it.
