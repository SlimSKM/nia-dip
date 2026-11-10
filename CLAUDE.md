# CLAUDE.md — Nia Document Import Platform (DIP)

DIP is a Django 5.2 + DRF + django-tenants service with a React SPA that
turns duty sheets into reviewed, standardised records and template
exports. It is interim and merges into TDH (ADR-DIP-001), so it follows
TDH's versions, names and patterns. Reference implementation (read only):
`/opt/nia/tdh/dev`.

**Precedence:** URD (`docs/requirements/DIP-URD-001`) > PDS
(`docs/architecture/DIP-PDS-001`) > phase prompt. Where they differ,
flag it; do not silently pick.

## Working rules for Claude Code

- Work one phase/step at a time on a feature branch (`p2-foundation`,
  ...). Never commit to or push `main` after the initial standards
  commit; never merge; never open a PR unless asked.
- One commit per step. Run `make test` (and the frontend tests once they
  exist) before every commit; never commit red.
- **Never `source`, `cat`, `grep`, `head` or otherwise print `.env`.**
  Give it to containers only through compose (`make ...`). Create it with
  `make env-init` (generates secrets in-process, never echoes them) and
  inspect it only with `make env-check` (key names and
  PRESENT / MISSING / EMPTY / SHELL-UNSAFE only). Report key names, never
  values.
- Never commit `.env`, secrets, or **any real client file** (no real
  PDFs/workbooks/CSVs; synthetic fixtures only under
  `backend/tests/fixtures/synthetic/`). Real fixtures live in
  `/opt/nia/dip/fixtures` (outside Git) and are read only by the opt-in
  `real_fixtures` suite after the real-data gate.
- Ask before adding any dependency not already approved in the phase
  prompt; pin exact versions. No PyMuPDF, ever (AGPL; ADR-DIP-002).
- Don't install system packages or change Docker daemon settings without
  asking.
- POPIA/GDPR: flag any new personal-data field before adding it. DIP keeps
  no driver identity (ADR-DIP-004). Soft delete is not erasure.
- Money (if it ever appears) is integer minor units + ISO 4217; no floats.
- Never write to the TDH checkout, never run its make targets, never
  touch `tdh-poc` or `ukw-poc`.
- At the end of each step: report files, tests, deviations, open
  questions. Stop at checkpoints.

## Naming and network (Nia group rule)

- Compose project `dip`; containers `dip_<role>` (`dip_web`, `dip_db`,
  `dip_migrate`, `dip_bootstrap`; later `dip_worker`, `dip_cache`);
  volumes/networks `dip_*`. Every make target passes `-p dip`.
- Every published port binds to **127.0.0.1** (web: 127.0.0.1:8020).
  Never publish on 0.0.0.0; no published DB port.
- Underscored container names are not valid HTTP Host headers: never call
  Django by its container name as the Host.

## Database roles and audit (TDH ADR-TDH-008 pattern)

- `dip_app` (runtime, the web process): DML only, owns nothing,
  append-only tables SELECT/INSERT only. `dip_owner` (non-superuser owner;
  CREATEDB/CREATEROLE only when bootstrapped with DEBUG): migrations,
  `apply_db_grants`, `provision_tenant`, tests — only in the one-shot
  `migrate` container. The container superuser only runs
  `bootstrap_db_roles` once. The web container never holds owner
  credentials.
- The audit log is hash-chained and written by DB triggers; never disable
  them outside a test. Management commands that write audited rows run
  inside `attributed()`.

## Code layout rules

- `backend/dip_core/` has **no Django, DRF or `apps.*` imports** (a test
  enforces it); it takes and returns plain data and moves to TDH
  unchanged.
- Django apps call `dip_core` and persist results. Provider specifics live
  in profile JSON, never in `if provider == ...` branches.
- API under `/api/v1`, errors as RFC 9457 problem+json with a DIP error
  code; JWT bound to the tenant.
- Tests run against PostgreSQL (never SQLite, except pure `dip_core` unit
  tests). Every DB test runs in a tenant schema via the tenant test base;
  no `TransactionTestCase` in tenant apps (its flush would TRUNCATE the
  audit log).

## Navigation rules (front end; TDH UX-NAV)

Breadcrumbs from the page hierarchy, never browser history; "← Back to
<parent>" on detail/edit/create pages; Cancel goes to a safe `next` or the
parent; `next` is accepted only if it is same-origin; no
`history.back()`; 403/404 pages link home (and to sign-in when signed
out).
