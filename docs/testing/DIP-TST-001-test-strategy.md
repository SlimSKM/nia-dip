**Nia Document Import Platform (DIP)**

**DIP-TST-001 — Test Strategy (Phase 1, step 1.8)**

| Field | Value |
|---|---|
| Version | 0.1 — draft for owner review |
| Date | 10 November 2026 |
| Basis | DIP-URD-001 v0.5, DIP-PDS-001 v0.1.1, DIP-FIX-001 v0.3, UKW-DEV-CICD-001, UKW-SEC-SCAN-001 |

## 1. Principles

- **Every requirement has a test.** Each Must and Should in DIP-URD-001 maps to at least one automated test, or a named manual UAT step. The traceability table lives in `docs/testing/traceability.md` and is kept current.
- **No silent data loss is tested, not assumed.** Every pipeline test asserts the reconciliation identities: units = processed + skipped, and rows = raw records, which = standard records + summary rows + failed rows.
- **Real data stays out of Git** (OQ-DIP-14). Git holds synthetic look-alikes only. The real Makhado files live in `/opt/nia/dip/fixtures` on `ukw-dev` (mode 700) and are used only by the opt-in regression suite.
- **PostgreSQL in every test run** (django-tenants, triggers, grants). SQLite only for pure `dip_core` unit tests.
- **Claims match reality.** Nothing is documented as working until its tests pass in CI.

## 2. Test levels and tools

| Level | Scope | Tools | Runs |
|---|---|---|---|
| Unit — core | `dip_core`: adapters, layout, redaction, mapping, normalisation, rules, reconciliation, dedup, export writer, crypto | pytest, hypothesis (property tests for time parsing and business keys) | Every commit, under 1 min |
| Unit — Django | Models, constraints, permissions, serializers | Django test runner / pytest-django | Every commit |
| Database | Triggers (append-only, immutability), grants (`dip_app` refused UPDATE/DELETE/TRUNCATE on protected tables), audit chain verification, CHECK constraints | pytest-django, with a throwaway role per test (the TDH M2 pattern) | Every commit |
| Integration | API endpoints end to end (upload → process → review → export), file store encryption, Celery tasks (Phase 4) | DRF test client, pytest; Celery eager mode plus one real-worker test | Every PR |
| Frontend | Components, forms, mapping table, review grid, approval rules, accessibility | Vitest, React Testing Library, axe-core | Every PR |
| End to end | Full workflow in a browser: Upload → Extract → Map → Validate → Review → Approve → Export → Report | **Playwright** (group standard), against the compose stack | Every PR to main; nightly |
| Regression — synthetic | Golden files: synthetic monitoring PDF, duty sheet and workbooks → expected standard records and expected export workbook (compared cell by cell and by SHA-256) | pytest | Every PR |
| Regression — real (opt-in) | The 8 Makhado sources + the Dolight target on `ukw-dev` only | pytest marker `real_fixtures`, path from environment | Manually and before each release candidate; never in GitHub Actions |
| Security | SAST, dependencies, secrets, plus targeted tests (§5) | Semgrep, Bandit, pip-audit, npm audit, Gitleaks, detect-secrets, OWASP Dependency-Check | Per UKW-SEC-SCAN-001 cadence |
| Performance | §6 targets | pytest-benchmark, Locust (API), timed fixture runs | Per phase gate and before release candidates |
| UAT | Owner-run scripts on `ukw-poc` | Checklist in `docs/testing/uat/` | Before POC sign-off |

**Coverage:** ≥ 70% lines overall (CI gate per UKW-DEV-CICD-001), and **≥ 90% for `dip_core`**, which TDH inherits.

## 3. Fixtures

### 3.1 Synthetic (in Git, `backend/tests/fixtures/synthetic/`)

A generator script (`make fixtures`) builds them deterministically from a seed:

| Fixture | Mimics | Built-in traps |
|---|---|---|
| `monitoring_timetable.pdf` (rendered, then rasterised and rotated 270°, with a watermark) | Makhado monitoring sheets | Sections across pages; header typo "Morning – Friday"; header variant "Route"; an unregistered route; signature-like scribbles outside the table; one sparse page with 1 row |
| `duty_sheet.pdf` (rotated 90°) | Dolight duty sheet | Subtotal rows; one trip ending before it starts; one empty Afternoon block; a route text that disagrees with its departure point |
| `operator_*.xlsx` (3 operators) | Operator workbooks | Header on row 1 and on row 7 under merged titles; "Monitoring Time" alias; an extra empty column; End Time formula missing on some rows; hidden sheet; formula cells |
| `identity_trap.xlsx` / `.csv` | — | Driver-name and SA ID columns (synthetic IDs with valid date and checksum) to prove redaction |
| `malicious/*` | — | Renamed executable, `.xlsm` with macro, zip bomb, encrypted PDF, PDF with JavaScript, path-traversal file name, 51 MB file |
| `target_template.xlsx` | Duty import template | Sample rows D-101/D-102; Lists sheet; data validation to 1001 |

Expected outputs sit beside each fixture (`*.expected.json`, `*.expected.xlsx`).

### 3.2 Real (outside Git)

| File | Assertions in the real-fixture suite |
|---|---|
| 6 operator workbooks | 610 raw records; 6 operators and 96 routes seeded; 45 Trip End failures (Phadziri G); 6 overlapping slot pairs in 5 duties reported; 610 → 582 rows after merge; contract reference VHE1 on every row |
| Monitoring Tools PDF | ≥ 95% of the 622 rows detected; OCR accuracy per field scored against the workbooks (the **Phase 4 acceptance gate**, §4); VHE1_108 and VHE1_060 rows reported as unassigned; 12 rows reported as missing from the workbooks when both sources are in one batch |
| Dolight duty sheet | 35 trips; subtotals checked; the 5 known issues reported (178 end before start, 170 route text, 172 departure, 174 count, 182 empty block) |
| Dolight target template | Sample rows removed; Lists and Instructions kept; data validation intact; output opens and passes the target's own rules (except fare matrix, OQ-DIP-17) |

Real fixtures are processed only after the real-data gate (NFR-DIP-012) is completed for the dev tenant. Until then the suite skips, with a clear message.

## 4. Key test cases (excerpt; the full catalogue is built in Phase 2)

| ID | Case | Requirement | Expected |
|---|---|---|---|
| TC-001 | Upload a renamed `.exe` as `.xlsx` | FR-DIP-011 | Rejected by content type, with a reason; batch continues |
| TC-002 | Same file uploaded twice | FR-DIP-012, 070a | Warning with a link to the first batch |
| TC-003 | Hidden sheet present | FR-DIP-013 | Listed as hidden; processed by default |
| TC-010 | Header on row 7 under merged cells | FR-DIP-020 | Parsed; header row recorded |
| TC-011 | End Time formula missing | FR-DIP-043 | Blank; Failed: core field missing; no derived value anywhere |
| TC-012 | Monitoring-PDF-only batch with an earlier approved workbook batch | FR-DIP-043 | Trip End pre-filled, `trip_end_source` = HISTORY, with a link |
| TC-020 | Identity columns present | NFR-DIP-010 | Values never stored (DB, logs, reports, exports grepped for the sentinels); redaction preview shown |
| TC-030 | Unregistered route on a multi-operator source | FR-DIP-037 | Failed: unassigned operator; row present and counted |
| TC-031 | OCR'd route code contradicts the From–To pair | FR-DIP-038 | Warning with a suggestion; not applied without the reviewer |
| TC-040 | Overlapping slots in one duty on one day | FR-DIP-087 | Export blocked; both records linked |
| TC-041 | Same trip Mon–Fri and Saturday | FR-DIP-086 | One export row with flags Mon–Sat; links to both records |
| TC-042 | Export the Dolight template | FR-DIP-081 | No sample rows; Lists/Instructions/validation kept; every row complete; times `hh:mm` |
| TC-043 | Re-export the same selection | FR-DIP-082, 083 | New file record, identical SHA-256; earlier file untouched |
| TC-044 | Contract column absent / present | FR-DIP-085 | VHE1 from the prefix / value from the column; Contract Route Y |
| TC-050 | Reviewer approves their own batch | FR-DIP-062 | Refused and audited |
| TC-051 | Administrator approves their own batch | FR-DIP-062 | Accepted; ★ self-approved on record, batch, reports, export history |
| TC-052 | Override an ERROR | FR-DIP-062 | Refused (database CHECK and API) |
| TC-060 | `dip_app` tries UPDATE/DELETE on raw records, approvals, exports, audit | FR-DIP-024, 100 | SQLSTATE 42501 |
| TC-061 | Audit chain tampered with by the owner role | FR-DIP-100 | `verify_audit_chain` fails and names the row |
| TC-070 | Originals purge after 30 days; hold set on one document | FR-DIP-101 | Held document kept; others purged; counts-only audit event; raw records intact |
| TC-080 | Read-only user calls every write endpoint | FR-DIP-003 | 403 on each (generated test over the URL map) |
| TC-090 | Batch totals reconcile on every fixture | NFR-DIP-001 | Identities hold |

**Phase 4 OCR acceptance gate** (on the real Monitoring Tools PDF, scored against the workbooks):
- rows detected ≥ 98%;
- Depart Time, Schedule Time, From and To ≥ 97% correct after reference checks;
- Route Code and Duty No. ≥ 95%;
- **0 wrong values marked high-confidence**: every wrong value must be flagged LOW_CONFIDENCE or carry a suggestion.

The last criterion matters most. Errors are acceptable only if DIP shows them to the reviewer.

## 5. Security tests

| Area | Tests |
|---|---|
| Authentication | Lockout after 5 failures; MFA required for Administrator and Reviewer; refresh rotation and reuse detection; JWT from another tenant rejected |
| Authorisation | Permission matrix (DIP-PDS-001 §8) checked per endpoint and role (generated); IDOR: records of another batch or tenant not reachable by id |
| Uploads | `malicious/*` fixtures; path traversal in file names; size and page limits; xlsx bomb caps; PDF JavaScript ignored; worker has no network (a test tries an outbound connection from the worker) |
| Injection / XSS | ORM-only check (Semgrep rule bans raw SQL with user input); cell values like `<script>`, `=HYPERLINK(...)`, `=cmd|...` survive as text and are written to exports as **strings** (CSV/Excel formula-injection guard: leading `= + - @` escaped in CSV and reports) |
| CSRF | Cookie-based refresh endpoint requires a CSRF token |
| Secrets | Gitleaks on history; `.env` never printed (no-print checker); logs grepped for tokens and redacted sentinels |
| Encryption | Files on the volume are ciphertext (no plaintext magic bytes); a wrong master key fails closed |
| Dependencies | pip-audit, npm audit and OWASP DC clean at High+ before each release candidate |

## 6. Performance tests (POC host profile: 6 vCPU shared, worker 2 vCPU / 2 GB)

| Scenario | Target (NFR-DIP-030) |
|---|---|
| XLSX, 10,000 rows | Staged in < 2 min |
| Text PDF, 100 pages | < 5 min |
| Scanned PDF | < 30 s per page (the real 61-page file < 30 min; prototype ≈ 7.5 min) |
| Review grid, 10,000 records | First page < 2 s |
| Export, 10,000 records | < 1 min |
| Bulk audit | Raw-record creation for 10,000 rows does not exceed the XLSX target (NFR-DIP-031) |
| Concurrency | 3 users uploading at once plus 1 OCR job: no errors; Specta, TDH and Moyo health checks stay green |

## 7. Environments and CI

- **GitHub Actions**, following the UKW-DEV-CICD-001 template:
  - Source → scan → build → unit and DB tests (PostgreSQL service container) → frontend tests → Playwright E2E (compose) → coverage gate.
  - Release candidates also run OWASP Dependency-Check and the performance smoke test.
- **`ukw-dev`:** the owner runs the real-fixture suite before each release candidate and records the result in the build log.
- **`ukw-poc`:** smoke test after each deploy (health, sign-in, synthetic upload → export), then UAT.

## 8. Defects and exit criteria

**Severities:** Critical (data loss, a security breach, wrong data exported as correct), High (a workflow blocked), Medium, Low.

**Exit criteria for each phase:**
- no open Critical or High defects;
- all Must requirements of that phase tested;
- coverage gates met;
- scans clean at High+;
- the phase's fixtures pass.

**POC sign-off** adds UAT by the owner, and the real-fixture suite green on `ukw-dev`.

## Change history

| Version | Date | Change |
|---|---|---|
| 0.1 | 10 Nov 2026 | First draft. |
