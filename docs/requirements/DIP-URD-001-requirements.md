**Nia Document Import Platform (DIP)**

**User Requirements Document**

DIP-URD-001 — Business, Functional and Non-Functional Requirements

| Field | Value |
|---|---|
| Document reference | DIP-URD-001 |
| Version | 0.5 — all open questions answered |
| Date | 10 November 2026 |
| Owner | Ukwazi Group (Siseko Peter) |
| Basis | DIP master project prompt (`duty_data.md`, updated 9 Nov 2026); DIP-DISC-001 v0.3 and the decisions of 9–10 Nov 2026; TDH-URD-OBR-001 Amendment A1 rev. 1 (v0.4) |
| Related | ADR-DIP-001 to 006; TDH ADR-TDH-015 |
| Status of fixtures | Eight source files supplied and analysed 10 Nov 2026 (DIP-FIX-001). Target `duty-import-template_dolight.xlsx` supplied and analysed the same day (DIP-FIX-001 §9); the owner confirmed it as **DIP's preferred output format**. |

## 1. Introduction

### 1.1 Purpose

This document states what DIP must do. DIP takes duty sheets supplied by transport service providers in different formats, extracts their data with full source traceability, standardises and validates it, puts it through human review and approval, and produces files in the layout of a chosen destination template. The first use case is the Makhado Bus Service duty data. The sources are the SPV's multi-operator monitoring timetable (scanned PDF), operator duty sheets (scanned PDF) and operator duty workbooks (xlsx). The target is the duty-import workbook `duty-import-template_dolight.xlsx` (Duties, Lists and Instructions sheets), confirmed by the owner as DIP's preferred output format.

### 1.2 Scope

**In scope (MVP):**
- XLSX, XLS and CSV import (Phase 3)
- text-based and scanned PDF import with OCR (Phase 4)
- versioned provider profiles and field mapping
- normalisation into a fixed standard duty schema
- validation and duplicate detection
- record-level review, correction and approval
- template-file and CSV export
- processing reports in Excel, CSV and PDF
- audit, retention settings and identity redaction
- one internal tenant

**Out of scope:** see §9.

### 1.3 Positioning

DIP is an interim service (ADR-DIP-001). It is built to merge into Transit Data Hub (TDH), front end included, and is retired after TDH milestone M10b. Requirements here are therefore a **subset of TDH-URD-OBR-001 Amendment A1 §5.9**. Appendix A traces each DIP requirement to its TDH counterpart. Where DIP is deliberately lighter than TDH (for example, it exports approved records directly instead of publishing a versioned Schedule Version), this is stated.

### 1.4 Definitions

| Term | Meaning |
|---|---|
| Provider | A transport service provider (operator) whose documents are imported. Held as a stable **operator code** that maps onto the TDH Operator at merge. |
| Provider profile | Versioned configuration for one provider and document type: sheet and page selection, header detection, field mapping, normalisation, validation rules, default export template. |
| Import batch | One or more files uploaded together for one provider. |
| Source document / source unit | An uploaded file, and a worksheet, page or table within it. |
| Raw record | One extracted row exactly as found (after identity redaction), with its source location. Never edited. |
| Standard record | A raw record mapped and normalised into the standard duty schema. Linked one-to-one to its raw record. |
| Discrepancy | A warning: the record can proceed if a reviewer accepts or overrides it with a reason. |
| Failure | An error: the record cannot proceed until it is corrected. |
| Export template | A destination layout, either a template workbook (e.g. Dolight) or a CSV column specification. |
| Export record | A standard record as written into one export file. |

### 1.5 References

DIP master project prompt; DIP-DISC-001 v0.3; ADR-DIP-001 to 006; TDH-URD-OBR-001 v0.3 and Amendment A1 rev. 1; UKW-DEV-CICD-001 v1.0; UKW-SEC-SCAN-001 v1.0; POPIA (Act 4 of 2013); GDPR (EU 2016/679).

## 2. Business context

Ukwazi Group and Gloss Data receive duty sheets from public-transport providers in whatever form each provider produces: workbooks with headers in odd places, PDFs and sometimes scans. Today these are re-keyed by hand into downstream import templates. That is slow and error-prone, and leaves no record of where each value came from. DIP replaces the re-keying with a traceable, reviewable pipeline. It must never claim an extraction succeeded when it didn't, and must never silently drop source data.

## 3. Stakeholders and roles

| Role | Can | Cannot |
|---|---|---|
| **Administrator** | Every action: upload, map, edit, approve and export (OQ-DIP-04), plus manage users, roles, reference data, profiles, templates, retention and holds, and download originals | — (self-approval is allowed but recorded and flagged; FR-DIP-062) |
| **Import operator** | Upload files; choose profile, template and units; map fields; correct records; resubmit; draft profiles | Approve records; change retention |
| **Reviewer / approver** | Review, accept, reject and approve records; override warnings with a reason; generate exports | Approve a batch they imported, or records they edited (FR-DIP-062) |
| **Read-only / reporting** | View batches, records, reports and export history | Change anything; download originals |

A user may hold several roles. Segregation of duties is enforced per batch and per record for Reviewers. Administrators may approve their own work; such approvals are flagged.

## 4. Assumptions and constraints

| # | Item |
|---|---|
| A-01 | Internal use only in the MVP. No provider logs in. |
| A-02 | One tenant (ADR-DIP-003). |
| A-03 | Two people are not always available (OQ-DIP-04). Administrators may therefore approve their own work; every such approval is flagged in the record, the batch and every report. |
| A-04 | Volumes (decided 10 Nov): up to 50 documents per batch, up to 10,000 rows or 100 PDF pages per document, daily batches. |
| A-05 | Data ownership (OQ-DIP-05): Makhado Bus Service is an SPV of operators contracted to the provincial department. The department, with the SPV administration, is the responsible party; Ukwazi processes on its behalf. All DIP users are internal. |
| A-06 | Operators are identified from the route register (FR-DIP-036), because the multi-operator monitoring PDF does not name them. |
| C-01 | Stack: Django + DRF, django-tenants, PostgreSQL 16, React + Vite + TypeScript, Docker Compose. No FastAPI, no PyMuPDF (ADR-DIP-002). |
| C-02 | Hosting in South Africa only (POPIA §72): `ukw-dev`, then `ukw-poc` behind the shared reverse proxy (`gdms_proxy` today; to be renamed `nia_proxy` and moved into its own stack in a late TDH sprint, Backlog item 17). Every port except the proxy binds to 127.0.0.1. |
| C-03 | No external AI services (D-11). OCR runs locally. |
| C-04 | No real personal data until the conditions in ADR-DIP-004 §4 are met. |
| C-05 | Original files only in the encrypted file store (ADR-DIP-006). |

## 5. Functional requirements

Priority: **M**ust, **S**hould, **C**ould. Phase: the build phase from the master prompt (3 Excel, 4 PDF, 5 approval and export, 6 profiles).

### 5.1 Access and administration

| ID | Requirement | Pri | Ph | Acceptance criteria |
|---|---|---|---|---|
| FR-DIP-001 | Users sign in with username and password; passwords are hashed (Django default, Argon2 or PBKDF2) and meet a password policy. Repeated failures lock the account temporarily. | M | 2 | Five failed sign-ins lock the account for 15 minutes; the lock is audited. |
| FR-DIP-002 | Users can enable TOTP two-factor authentication. Before any real client data is loaded, it is **required** for Administrators and Reviewers. | S (M before real data) | 2 | With the "require MFA" setting on, an Administrator can't reach any page without TOTP. |
| FR-DIP-003 | Role-based access as in §3, enforced on every API endpoint and screen. Defaults are least-privilege. | M | 2 | A Read-only user gets 403 on every write endpoint (a test covers every endpoint). |
| FR-DIP-004 | An Administrator manages users and roles in the tenant. Tenant accounts can never be superuser or staff. | M | 2 | — |

### 5.2 Upload and batches

| ID | Requirement | Pri | Ph | Acceptance criteria |
|---|---|---|---|---|
| FR-DIP-010 | An import operator uploads one or more files by drag-and-drop or file picker into a new **import batch**, chooses a provider and profile (or confirms the suggested one) and an export template. | M | 3 | A batch of 3 files for one provider is created with all three listed. |
| FR-DIP-011 | Each file is checked before processing: type by **content**, not extension (XLSX, XLS, CSV, PDF); size up to **50 MB per file** and **500 MB per batch** (the monitoring PDF alone is 22.7 MB); up to 100 PDF pages; not encrypted or password-protected; no macros run. A rejected file is listed with its reason. Accepted files are never silently skipped. | M | 3 | A renamed `.exe` and a 30 MB file are both rejected with reasons; the others are processed. |
| FR-DIP-012 | The SHA-256 hash of each file is recorded on upload. Uploading a file whose hash is already in the tenant warns the user before processing and links to the earlier batch (FR-DIP-070). | M | 3 | Re-uploading the same file shows the warning. |
| FR-DIP-013 | The batch lists every **source unit** (worksheet, page or table), including hidden worksheets, flagged as hidden. **All units are processed by default**. The user may choose specific units or a custom subset, and skipped units are recorded as skipped. | M | 3 | The batch record shows every unit and whether it was processed or skipped. |

### 5.3 Extraction

| ID | Requirement | Pri | Ph | Acceptance criteria |
|---|---|---|---|---|
| FR-DIP-020 | **Spreadsheets:** read every selected sheet. Detect header rows without assuming row 1. Handle blank rows, merged header cells, several tables per sheet, and inconsistent column names. Read formulas as their cached values, and show that limitation. Keep the source cell type (text, number, date, time). | M | 3 | A sheet whose header starts on row 7 under merged title cells parses correctly; a formula cell shows its cached value with a "formula" marker. |
| FR-DIP-021 | **CSV:** detect delimiter and encoding (UTF-8, UTF-8 with BOM, Windows-1252); report the detected values. | M | 3 | A semicolon-separated Windows-1252 file parses correctly. |
| FR-DIP-022 | **Text PDFs:** extract tables per page with `pdfplumber`, recording page and table identifiers. Rotated pages are corrected first. A table that continues across pages is joined into one table, with page references kept per row. | M | 4 | A monitoring section spanning pages 1–3 becomes one table; every row knows its page. |
| FR-DIP-023 | **Scanned PDFs:** pages with no text layer are OCR'd with Tesseract. Every cell and row gets a confidence score. Handwritten content is not interpreted and is reported as unreadable. | S | 4 | A scanned page yields rows with confidence scores; a handwritten entry is reported, not treated as empty. Signatures and stamps outside the table are not extracted. |
| FR-DIP-026 | **Section context.** Values printed once per page or block (monitoring point, time of day, period week, operator name in a page header, Morning/Afternoon block) are captured with the unit and carried onto every row in it, as raw values. Header labels are matched by text, allowing per-profile variants (e.g. "Route" or "Route Code"; "Schedule Time" or "Monitoring Time"). | M | 3/4 | On the monitoring PDF, every row carries its page's monitoring point, time and period week. |
| FR-DIP-027 | **Page orientation** is detected per page (orientation detection, not the PDF rotate flag alone), because sources in one batch are rotated differently. | M | 4 | The monitoring PDF and the Dolight duty sheet, rotated opposite ways, both read upright. |
| FR-DIP-028 | **Subtotal and summary rows** (e.g. "Sub total kilometers") are recognised per profile. They are kept as raw records marked "summary", excluded from standard records, and used for consistency checks. | M | 4 | Duty-sheet subtotals don't become trips; a subtotal that disagrees with its rows raises a discrepancy. |
| FR-DIP-024 | Every extracted row is stored as a **raw record** before mapping: original values and column names (after redaction, NFR-DIP-010), file name and hash, sheet name and row number, page, table id, extraction method (cell, CSV, PDF text, OCR), confidence, extraction time, batch, provider and profile version. Raw records are never edited or deleted by the pipeline. | M | 3 | From any export record, a user can trace back to the file, sheet or page, and row. |
| FR-DIP-025 | Extraction that is **incomplete or uncertain** is reported as such: below-threshold confidence, unparsed regions, or a table cut off mid-page. A batch with uncertain content is never shown as "Successful". | M | 3/4 | A low-confidence OCR cell is highlighted, and the batch shows "Needs review". |

### 5.4 Provider profiles and mapping

| ID | Requirement | Pri | Ph | Acceptance criteria |
|---|---|---|---|---|
| FR-DIP-030 | **Provider profiles** per provider and document type hold: unit selection rules, header detection settings, PDF and table-detection settings, OCR settings, field mapping, normalisation rules, validation rules, confidence threshold (an integer, per mille) and default export template. Profiles are validated against a JSON Schema. | M | 3 (basic) / 6 (full) | An invalid profile is rejected with the failing rule named. |
| FR-DIP-031 | Profiles are **versioned**. Editing creates a new draft version; an Administrator approves it before use (maker-checker). Every batch records the profile version it used. | M | 6 | An old batch still shows its original profile version after the profile changes. |
| FR-DIP-032 | Mapping **suggestions** come from name similarity, mappings confirmed earlier and a layout fingerprint. The user confirms or changes each one. A file whose fingerprint matches a profile is recognised automatically. | M | 3 | A second file with the same layout needs no remapping. |
| FR-DIP-033 | **Unmapped columns are kept** in the raw record and shown in review, never discarded. | M | 3 | A new column appears as "unmapped". |
| FR-DIP-034 | A mapping confirmed during a batch can be **saved to the profile** as a new draft version. | M | 6 | — |
| FR-DIP-035 | Profiles have a **source type**: *monitoring timetable* (multi-operator, sections by monitoring point), *operator duty sheet* (one duty per page, Morning and Afternoon blocks) or *operator duty workbook*. The Makhado fixtures need all three. | M | 3/4 | One profile of each type processes its fixture. |
| FR-DIP-036 | **Reference data**, versioned and maintained by Administrators: **operators** (stable operator code); **route register** (code, operator, From, To, description, service type, status such as cancelled); **place list** with aliases; **monitoring points**. It can be seeded from operator workbooks and route lists, with a preview before saving. | M | 3 | Seeding from the six workbooks gives 6 operators and 96 routes, each with one operator. |
| FR-DIP-037 | **Operator attribution.** On multi-operator sources, each row's operator comes from its route code via the route register. A route that isn't registered gives processing status **Failed: unassigned operator**. The row is kept and shown, never dropped or guessed. | M | 3 | Rows for VHE1_108 (not in any workbook) are listed as unassigned, not lost. |
| FR-DIP-038 | Reference data **supports extraction**: OCR'd place names are matched to the place list, and route codes are checked against the From–To pair in the route register. Any correction is a *suggestion* with its confidence, shown beside the OCR'd value. | S | 4 | An OCR'd "VHE1_004" with From–To Maila Lagos → Makhado is flagged, with VHE1_001 suggested. |

### 5.5 Standard schema and normalisation

| ID | Requirement | Pri | Ph | Acceptance criteria |
|---|---|---|---|---|
| FR-DIP-040 | The **standard duty record** has four **core fields**, which must be populated before approval: **Duty No.** (→ Duty Name), **Route** (route code, plus From → To), **Trip Start** (→ Start Time), **Trip End** (→ End Time). It also has **target fields**: operator (→ Depot), **day flags** Sun–Sat (from day type), Status, Approval Status, Contract Route and Contract Reference. And it has **context fields** kept for traceability and checks: day type, time period, monitoring point, From, To, schedule (monitoring) time, source row reference, Rev kms, Positioning. Context fields may be blank in the output (owner, 10 Nov). | M | 3 | Every standard record shows its core, target and context values with their sources. |
| FR-DIP-043 | **Trip End is copied, never derived** (owner, 10 Nov). Sources, in order: (1) the record's own source (the duty sheet's *Trip end*, or the workbook's *End Time* cached value); (2) another document in the same batch that matches on operator, day type, duty and trip start; (3) **the latest approved record from an earlier batch** with the same operator, day type, duty, route, From and trip start (OQ-DIP-22). The source used is recorded on the record (`trip_end_source`) and shown in review. If none applies (the case for monitoring-PDF-only batches with no history), the record is **Failed: core field missing** until a reviewer enters it. | M | 3 | A workbook row without End Time stays blank and fails; a monitoring-PDF row whose trip was approved in an earlier workbook batch gets that End Time, labelled with its source; no +10-minute value appears anywhere. |
| FR-DIP-041 | Normalisation rules per profile: trim and case rules; code lists (e.g. route aliases, and header typos such as "Morning – Friday" → Monday – Friday); day-type values; date formats; time formats including `04H15`, `04:15`, Excel time values and times **past 24:00** (stored as minutes after the start of the service day); time zone Africa/Johannesburg. Values that can't be normalised become failures, not blanks. | M | 3 | "25:10" is accepted as 1,510 minutes; "7.5" with no matching time format is a failure. |
| FR-DIP-042 | The standard record keeps the original value beside the normalised value for every mapped field. | M | 3 | The review screen shows both. |

### 5.6 Validation and statuses

| ID | Requirement | Pri | Ph | Acceptance criteria |
|---|---|---|---|---|
| FR-DIP-050 | A configurable rule set runs on standard records: required fields, data types, date and time validity, field formats (e.g. duty number pattern), cross-field rules (trip end after trip start, allowing past-24:00 times), business rules from the profile, references (route in the provider's route list), source consistency (row count against totals on the sheet, where present), and **export-template compatibility** (every required template field can be filled). | M | 3 | Each result names the rule, field, severity and source location. |
| FR-DIP-055 | **Cross-source reconciliation.** Batches normally hold one source format (owner, 10 Nov). When a batch does hold more than one source for the same operator (e.g. monitoring PDF, duty sheet and workbook), DIP compares them per operator, day type and duty: trip count, trip starts, From, route, and trip end ≥ trip start. It also reports source rows missing from another source (e.g. monitoring-PDF rows absent from every workbook). Differences are discrepancies linked to both sources. **There is no automatic winner** (OQ-DIP-13). | M | 5 | On the fixtures, all five Dolight issues and the 12 monitoring-PDF rows missing from the workbooks are reported (DIP-FIX-001 §4, §5). |
| FR-DIP-051 | Each rule has a severity: **error** (failure) or **warning** (discrepancy), set per profile. | M | 3 | — |
| FR-DIP-052 | Each standard record carries **three separate statuses**. **Processing:** Successful / Successful with discrepancies / Failed. **Review:** Pending review / Approved / Rejected. **Export:** Not exported / Exported (with file references). No single field mixes them. | M | 3/5 | A record can be "Successful with discrepancies", "Approved" and "Not exported" at once. |

### 5.7 Review and correction

| ID | Requirement | Pri | Ph | Acceptance criteria |
|---|---|---|---|---|
| FR-DIP-060 | The review screen shows, per record: source location (with a link to the raw record and, for PDFs, the page image), original values, normalised values, validation results, confidence and statuses. It can be filtered by status, rule, unit and confidence. | M | 3 | — |
| FR-DIP-061 | Operators can **edit** normalised values to correct failures, and **reprocess** records or units after a profile change. The raw record never changes. Every edit records who, when, the old value, the new value and a reason. | M | 3 | An edited record shows its edit history; the raw record is unchanged. |
| FR-DIP-062 | Reviewers can **approve** or **reject** records singly, in bulk, or for the whole batch, and **override warnings** with a mandatory reason. Errors can't be overridden, only corrected. **A Reviewer who imported a batch, or edited a record, can't approve that record.** An **Administrator** may (OQ-DIP-04). The approval is then marked *self-approved* on the record and batch and in every report and export history. | M | 5 | A Reviewer's own-approval attempt is refused and audited; an Administrator's is accepted and flagged. |
| FR-DIP-063 | **Partial processing:** approved records can be exported while failed or pending ones stay in the batch for later correction. Failed records stay available indefinitely (subject to retention). | M | 5 | A batch of 100 rows with 5 failures exports 95; the 5 can be fixed and exported later. |

### 5.8 Duplicates

| ID | Requirement | Pri | Ph | Acceptance criteria |
|---|---|---|---|---|
| FR-DIP-070 | Duplicate checks at five levels, each switchable per profile: (a) same file (hash); (b) same source row processed twice; (c) duplicate records within a document (by business key: **operator + day type + duty no. + route + From + trip start**, revised from OQ-DIP-08 because the shorter key collides 37 times in the fixtures); (d) duplicates across batches (same business key in an earlier approved batch); (e) duplicate exports (a record already exported with the same template). | M | 3 (a–c) / 5 (d–e) | Each level raises a "Possible duplicate" with a link to the other record or batch. |
| FR-DIP-071 | Duplicates are **never deleted automatically**. A reviewer decides: keep both, skip this one, or replace the earlier one in a new export. The decision and reason are audited. | M | 3 | — |

### 5.9 Export

| ID | Requirement | Pri | Ph | Acceptance criteria |
|---|---|---|---|---|
| FR-DIP-080 | The **default export template** is the duty-import workbook (DIP-FIX-001 §9): sheet *Duties*, 16 columns (Duty Name, Depot, Route, Start Time, End Time, Sun–Sat, Status, Approval Status, Contract Route, Contract Reference), with the *Lists* and *Instructions* sheets. Templates are uploaded per target tenant, versioned and approved; the column mapping from the standard record is configured, not hard-coded. | M | 5 | The Dolight template is configured without code changes. |
| FR-DIP-081 | **Template-file export** writes into a copy of the uploaded template with openpyxl. It **removes the sample rows** (the target imports every row left), keeps *Lists*, *Instructions*, styles, column widths and data validation, and extends the validation ranges if more than 1,000 rows are written. Parts that can't be kept are listed at upload and must be accepted (as before). Every field is written on **every row**, never relying on the target's "blank = row above" rule. Times are written as time values in `hh:mm` format. | M | 5 | The export opens with the template's formatting and dropdowns intact, no sample rows, and every row complete. |
| FR-DIP-082 | Exports are **reproducible**: workbook metadata and archive timestamps are pinned, so the same records and template version give the same file hash. | S | 5 | Two exports of the same selection have identical hashes. |
| FR-DIP-083 | The reviewer selects approved records (by batch, filter or explicit selection), chooses a template, generates the file, and downloads it. Each generation creates an **export file** record: template version, record count, file hash, who and when, and links to every export record. Earlier exports are **never overwritten**; a new export is a new file. | M | 5 | Export history lists every file, and re-downloading gives an identical file. |
| FR-DIP-084 | **Target master data (informational).** On template upload, DIP reads the *Lists* sheet as a dated snapshot of the target's depots and routes. Depot and route names not in the snapshot are shown as **warnings**, not failures; the fare-matrix column is ignored (OQ-DIP-17). | S | 5 | After upload, the snapshot is listed with its date; unknown names appear as warnings in review. |
| FR-DIP-085 | **Target mapping.** **Duty Name** ← duty no. (plain; optional per-operator prefix, OQ-DIP-15). **Depot** ← the operator's depot: each operator has one or more depots in reference data. When a source gives no depot, the operator's **default depot** is used; when an operator has several, the depot can be chosen per batch at upload and changed per record during review (OQ-DIP-16). **Route** ← the route register's target route name per direction (default pattern "{From} to {To}", overridable per route). **Day flags** ← day type. **Status** ← Active; **Approval Status** ← **Approve** (export defaults, changeable per export; OQ-DIP-18). **Contract Reference** ← a contract column in the source when present (header *contract*, *contract_code* or *contract_no*, matched by the profile); otherwise **the part of the route code before the first underscore** (e.g. VHE1_001 → VHE1). **Contract Route** ← Y when a Contract Reference was found, otherwise N (OQ-DIP-19). | M | 5 | A Dolight row VHE1_001 Makhado → Maila Lagos, Monday–Friday, maps to Depot = Dolight's default depot, Route "Makhado to Maila lagos", Mon–Fri = Y, Status Active, Approval Status Approve, Contract Route Y, Contract Reference VHE1. |
| FR-DIP-086 | **Merge across day types.** Records of the same operator, duty, route, From, To, start and end that differ only in day type become **one row** with the combined day flags, as in the template's own sample. Switchable per export (default on); each export row keeps links to all its records. | S | 5 | On the fixtures, 610 records give 582 rows. |
| FR-DIP-087 | **Export-compatibility validation** checks the target's import rules before export, as failures by default (the target would reject the duty): Duty Name present, ≤255 characters and unique per operator within the export; not already exported to the same target (from export history); Depot set; Route set; Start and End present, **End after Start**; **no two slots of one duty overlap on the same day** (back-to-back allowed); at least one day flag; first-row values (Depot, Status, Approval) consistent across a duty; Contract Reference present when Contract Route = Y. **The target's fare-matrix rule is not applied** (it belongs to the target platform, not DIP; OQ-DIP-17). | M | 5 | On the fixtures, the 45 missing End Times and the 6 overlapping slot pairs in 5 duties are reported before any file is generated (DIP-FIX-001 §9). |
| FR-DIP-088 | **Target-readiness report.** For a batch or export, DIP lists depot and route names that aren't in the target's last snapshot, with the affected duties and record counts, so they can be set up in the target system. Informational. Available in XLSX, CSV and PDF. | S | 5 | On the fixtures it lists the Makhado depots and the 166 route names not yet in the snapshot. |
| FR-DIP-089 | **Output split.** One file per operator (default) or one combined file, chosen per export (OQ-DIP-20). | S | 5 | — |

### 5.10 Reports

| ID | Requirement | Pri | Ph | Acceptance criteria |
|---|---|---|---|---|
| FR-DIP-090 | **Per-batch processing report:** batch id, provider, source documents with hashes, processing time, totals (all, successful, with discrepancies, failed, approved, rejected, exported), discrepancy details, overrides with reasons and approvers, and source references. | M | 5 | — |
| FR-DIP-091 | **Failure report:** batch id, document, record id, sheet, page and row, rule, failure reason, relevant source values (redacted where needed), and the suggested correction where the rule defines one. | M | 5 | — |
| FR-DIP-092 | **Other reports:** processing summary over a date range; duplicates; records awaiting review; rejected; approved; export history; audit history (filterable). | S | 5 | — |
| FR-DIP-093 | Every report is available in **XLSX, CSV and PDF** (openpyxl, `csv`, ReportLab 4.4.10), with provenance and processing metadata in a header block. Reports never contain redacted identity values. | M | 5 | Each report opens in all three formats; a test asserts that no redacted value appears. |

### 5.11 Dashboard

| ID | Requirement | Pri | Ph | Acceptance criteria |
|---|---|---|---|---|
| FR-DIP-095 | The dashboard shows recent batches by status, failures, records pending review, recent exports, and simple processing statistics (batches and records per week, failure rate). | S | 5 | — |

### 5.12 Audit and retention

| ID | Requirement | Pri | Ph | Acceptance criteria |
|---|---|---|---|---|
| FR-DIP-100 | An **append-only audit log** records: sign-ins and failures, uploads, profile and template changes and approvals, mapping confirmations, edits, overrides, approvals and rejections, duplicate decisions, exports and downloads, original-file access, retention changes, holds and purge runs. It is enforced in PostgreSQL with triggers and a hash chain, following the TDH M2 pattern. The application role can't update or delete audit rows. | M | 2/3 | An UPDATE or DELETE on the audit table as the application role fails; `verify_audit_chain` passes. |
| FR-DIP-101 | **Retention settings** per data class (originals, raw records, standard records, export records and files, reports, audit records) are visible to Administrators. Only the **originals purge** (default 30 days, with a hold option) is switched on in the MVP. The others are recorded settings without automated deletion until their rules are approved (ADR-DIP-004). | M | 5 | Changing a setting is audited; the purge job writes a counts-only event. |
| FR-DIP-102 | **Controlled deletion:** an Administrator can delete a batch's original files early, with a reason. This is audited. Records stay. | S | 5 | — |

## 6. Non-functional requirements

| ID | Requirement |
|---|---|
| NFR-DIP-001 | **No silent data loss.** Every uploaded file, unit and row is accounted for as processed, skipped (with a reason), rejected (with a reason) or failed. Batch totals reconcile: units = processed + skipped; rows = raw records. |
| NFR-DIP-002 | **Traceability.** Every export record links to its standard record, raw record, source unit and source document (by hash), together with the profile and template versions. |
| NFR-DIP-010 | **Identity redaction (POPIA §10, GDPR Art. 5(1)(c)).** As ADR-DIP-004 §1. Profiles declare identity columns. Detection also flags likely names in "driver" or "employee" columns, employee-number patterns and South African ID numbers (13 digits with a valid date and checksum) anywhere in unmapped content. Redaction happens before storage. A preview is shown before confirmation. Tests use synthetic identity data only. |
| NFR-DIP-011 | **Originals.** Encrypted at rest (ADR-DIP-006), kept 30 days by default, with a hold option, a purge job and audited access (ADR-DIP-004 §2). |
| NFR-DIP-012 | **Lawful processing gate.** Real client documents are refused (a tenant setting "real data enabled", off by default) until an Administrator records: the responsible party (for Makhado: the provincial department with the SPV administration, A-05), the lawful basis, the DPA reference for Virtarix, and the date the redaction test passed on a sample. The Makhado sources contain no driver identity, but the monitoring PDF carries officials' signatures in its stored original. |
| NFR-DIP-013 | **Data residency (POPIA §72).** All storage and processing in South Africa; no external AI or OCR services. |
| NFR-DIP-020 | **Upload security.** Content-type sniffing; size and page limits (FR-DIP-011); XLSX parsed with `defusedxml` protection; archive-bomb limits on XLSX (uncompressed size and entry count); macros never executed; PDF parsing in a worker with no network (Phase 4); random storage names; uploads streamed to encrypted storage, never to plain temp files. |
| NFR-DIP-021 | **Application security.** Covers the OWASP Top 10, including: ORM-only queries (no raw SQL with user input); output encoding in React (no `dangerouslySetInnerHTML` on data); CSRF protection for cookie-based auth; secure, HttpOnly, SameSite cookies; HSTS and the group security headers at the proxy; rate limits on sign-in and upload endpoints; generic error messages to users, details only in logs; no secrets in logs or reports. |
| NFR-DIP-022 | **Secrets.** Only in `.env`, never committed. `.env` is never printed (CLAUDE.md rule). A no-print key checker, as in TDH. Separate database roles for owner and runtime, as in TDH M2. |
| NFR-DIP-023 | **Scanning.** UKW-SEC-SCAN-001 tools in CI (Semgrep, Bandit, pip-audit, npm audit, Gitleaks, detect-secrets; OWASP Dependency-Check before release). The required repo files are present from Phase 2. |
| NFR-DIP-030 | **Performance (Medium volume, POC host).** XLSX of 10,000 rows staged in under 2 minutes; text PDF of 100 pages in under 5 minutes; scanned page in under 30 seconds (prototype: about 7 s per page; DIP-FIX-001 §6.7); review screen for 10,000 records paginated with first page under 2 seconds; batch export of 10,000 records in under 1 minute. Worker capped at 2 vCPU and 2 GB; other products on the host unaffected. |
| NFR-DIP-031 | **Bulk audit.** Audit writes for bulk raw and standard records must not break NFR-DIP-030. Audit is at batch level for raw-record creation (counts and hashes), and at record level for every human action. Throughput is measured in Phase 3 (TDH risk R-3). |
| NFR-DIP-040 | **Availability.** POC: best effort during business hours. Daily database backup and an encrypted-file backup with a tested restore (operations guide). |
| NFR-DIP-041 | **Portability.** Docker Compose. One process per container. Configuration through environment variables. Services can be mapped 1:1 to Helm charts. No cloud-specific dependency. |
| NFR-DIP-050 | **Merge readiness.** Entity names as in TDH OBR-001 A1 §5; API style as TDH M5 (versioned `/api/v1`, JWT bound to the tenant, RFC 9457 errors); React feature modules without their own shell; `dip_core` with no Django imports; same Django and django-tenants versions as TDH. |
| NFR-DIP-051 | **Code quality.** PEP 8, type hints, Ruff and Black (or Ruff format), mypy on `dip_core`, ESLint and Prettier on the front end, ≥70% line coverage (UKW-DEV-CICD-001), structured JSON logging, reversible migrations. |
| NFR-DIP-052 | **Accessibility and UI.** Responsive down to tablet width; keyboard-operable review table; WCAG 2.1 AA contrast; TDH navigation rules (breadcrumbs, back-to-parent, validated `next`, safe cancel). |

## 7. Data requirements (logical)

Names follow TDH-URD-OBR-001 A1 §5 (NFR-DIP-050). The physical model and ERD are in DIP-PDS-001 (Phase 1 step 1.5).

| Entity | Key attributes |
|---|---|
| Provider (Operator) | Operator code (stable), name, status |
| Depot | Operator, name (as in the target), default flag (one default per operator) |
| Route | Code, operator, From, To, description, service type, status, version |
| Place | Name, aliases |
| Monitoring Point | Name, aliases |
| Provider Profile (versioned) | Provider, document type, version, settings (JSON, schema-validated), status, approved by and at |
| Import Batch | Provider, profile version, export template version, created by and at, unit selection, counts by status |
| Source Document | Batch, file name, SHA-256, size, detected type, storage key, encrypted flag, retention-until, hold, purged-at |
| Source Unit | Document, kind (sheet, page or table), name or number, hidden, rotation, processed or skipped (with reason) |
| Raw Record | Unit, row and page reference, original column names and values (redacted), extraction method, confidence |
| Standard Record | Raw Record (1:1), standard field values with original values, three statuses, confidence |
| Validation Result | Standard Record, rule, severity, message, overridden by and at, reason |
| Record Edit | Standard Record, field, old value, new value, who, when, reason |
| Duplicate Candidate | Records or batches involved, level, decision, decided by and at, reason |
| Approval | Standard Record or batch, decision, who, when, override reasons |
| Export Template (versioned) | Name, kind (workbook or CSV), template file reference and hash, mapping, unsupported-parts list and acceptance, version, status |
| Export File | Template version, record count, file hash, storage key, who, when |
| Export Record | Export File, Standard Record, row position |
| Processing Report | Batch or date range, type, format, file hash, who, when |
| Audit Log Entry | Hash-chained, append-only |
| Retention Setting | Data class, period, automated (yes or no), changed by and at |
| Original File Access / Purge Run | Insert-only event tables |

## 8. Interfaces

- **Inbound:** file upload through the web UI and API only. No email, FTP or system-to-system feeds.
- **Outbound:** file download (exports and reports). No push to Dolight or any other system in the MVP.
- **API:** REST under `/api/v1`, documented in OpenAPI 3.1 (DIP-PDS-001). This is the draft of the TDH M5 import-resource contract.
- **Internal:** no dependency on TDH at run time.

## 9. Out of scope (MVP)

- handwriting recognition
- AI-assisted extraction (D-11; an extension point only)
- provider self-service logins
- more than one tenant, billing or licensing
- system-to-system delivery to Dolight
- editing source files
- automated deletion of anything except originals
- timetable and fare-sheet imports (TDH FR-OBR-098; later)
- named driver assignments (TDH M12; DIP never stores driver identity)

## 10. Open questions

| ID | Question | Status |
|---|---|---|
| OQ-DIP-01 | Makhado duty sheet | **Answered 10 Nov.** Multi-operator monitoring PDF, plus one workbook per operator; no driver identity. Analysed in DIP-FIX-001. |
| OQ-DIP-02 | Dolight template | **Answered.** All earlier files are sources. The target is `duty-import-template_dolight.xlsx`, supplied 10 Nov; it is DIP's preferred output format. |
| OQ-DIP-03 | Who is Dolight? | **Answered.** A transport operator, and the owner of the template. |
| OQ-DIP-04 | Two-person rule | **Answered.** Not always possible. Administrators may do every action; self-approval is flagged (FR-DIP-062). |
| OQ-DIP-05 | Responsible party | **Answered.** The department with the SPV administration (A-05). All users internal. |
| OQ-DIP-06 | Retention beyond originals | **Approved.** Records and reports 2 years, audit 5 years; recorded, not automated in the MVP. |
| OQ-DIP-07 | MFA timing | **Approved.** From the first POC deploy for Administrators and Reviewers. |
| OQ-DIP-08 | Duplicate business key | **Approved, then revised** by the fixture evidence (FR-DIP-070). |
| OQ-DIP-09 | Target template columns and core-field mapping | **Answered:** FR-DIP-080 to 089; DIP-FIX-001 §9. |
| OQ-DIP-10 | Trip End for monitoring-PDF-only trips | **Recommendation in FR-DIP-043:** fill from a matching duty sheet or workbook in the batch, otherwise fail for review. |
| OQ-DIP-11 | Operator for routes in no workbook, and route-register ownership | **Accepted:** the SPV administration owns the route register; a DIP Administrator maintains it; the owner supplies the operator for VHE1_060, VHE1_108 and the other unregistered codes. Until then those rows fail as *unassigned operator*. |
| OQ-DIP-12 | Day type of duty-sheet pages with no date | **Accepted:** set per document at upload, default Monday–Friday. |
| OQ-DIP-13 | Which value is pre-filled when sources disagree | **Accepted.** Normally a batch holds one source format; when it holds several: workbook, then monitoring PDF, for timetable fields; duty sheet for Trip End; every difference is a discrepancy. |
| OQ-DIP-14 | Storing the supplied files as fixtures | **Accepted:** outside Git on `ukw-dev`; synthetic look-alikes in Git. |
| OQ-DIP-22 | Trip End when a batch holds only the monitoring PDF | **Answered:** pre-fill from the latest approved record of an earlier batch with the same operator, day type, duty, route, From and trip start; otherwise the record fails for review (FR-DIP-043). |
| OQ-DIP-15 | Duty Name format | **Accepted:** plain duty no., optional per-operator prefix. |
| OQ-DIP-16 | Depot mapping | **Answered:** an operator has one or more depots. Default depot when none given; choose at upload or map manually in review when an operator has several (FR-DIP-085). |
| OQ-DIP-17 | Target routes and fare matrices | **Answered:** the fare-matrix rule belongs to the target platform and is not applied in DIP. Unknown target names are warnings only (FR-DIP-084, 088). |
| OQ-DIP-18 | Status and Approval Status defaults | **Answered:** Active and **Approve**. |
| OQ-DIP-19 | Contract Route / Reference | **Answered:** reference from a source contract column (*contract*, *contract_code*, *contract_no*) if present, else the route-code prefix before "_"; Contract Route = Y when a reference is found. |
| OQ-DIP-20 | Output split | **Accepted:** one file per operator by default. |
| OQ-DIP-21 | End Time and overlaps | **Accepted:** copy from source; pre-fill from the duty sheet's Trip end where present; overlaps fail for review. |

## 11. User stories (MVP)

| ID | Story | Covers |
|---|---|---|
| US-01 | As an import operator, I upload a provider's duty sheets and see every sheet and page listed, so nothing is missed. | FR-DIP-010 to 013 |
| US-02 | As an import operator, I map the provider's columns to the standard fields once and have later files recognised automatically. | FR-DIP-030 to 034 |
| US-03 | As an import operator, I see which rows failed and why, with the source location, and correct them without losing the original value. | FR-DIP-050 to 061 |
| US-04 | As a reviewer, I approve good records, override warnings with a reason, and leave failures behind, so a partly correct batch still moves forward. | FR-DIP-062, 063 |
| US-05 | As a reviewer, I'm warned when a sheet or record has been imported before, and I decide what to do with it. | FR-DIP-070, 071 |
| US-06 | As a reviewer, I generate the duty-import workbook from approved records, without sample rows, with every row passing the target's rules. | FR-DIP-080 to 087 |
| US-12 | As an Administrator, I see which depot and route names the target system doesn't know yet, before the file is imported. | FR-DIP-084, 088 |
| US-07 | As a manager, I download a batch report showing totals, discrepancies, overrides and failures, with source references. | FR-DIP-090 to 093 |
| US-08 | As an Administrator, I can show an auditor who imported, changed, approved and exported each record, and that no driver identity was kept. | FR-DIP-100, NFR-DIP-010 |
| US-09 | As an import operator, I upload a scanned duty sheet and see which values the OCR was unsure of. | FR-DIP-023, 025 |
| US-10 | As an import operator, I upload the SPV's monitoring timetable and get every row assigned to its operator, with unregistered routes listed, not lost. | FR-DIP-035 to 037 |
| US-11 | As a reviewer, I see where an operator's duty sheet disagrees with the monitoring timetable or its workbook, and decide which is right. | FR-DIP-055 |

## 12. Acceptance (definition of done for the MVP)

- Every Must requirement is implemented and demonstrated on the regression fixtures (synthetic, plus the Makhado sample once authorised).
- The end-to-end test passes: upload → extract → map → validate → review → approve → export → report.
- Security scans are clean at High and above; the redaction tests pass; the audit chain verifies.
- Docs match the implementation: features are documented only once built and tested.
- The owner signs off UAT on `ukw-poc`.

## Appendix A — Traceability to TDH-URD-OBR-001 Amendment A1

| DIP | TDH OBR A1 | Note |
|---|---|---|
| FR-DIP-010 to 013 | FR-OBR-080 to 082, 093 (file level) | Same |
| FR-DIP-020 to 028 | FR-OBR-081, 083, 084, 088 | Same; section context and orientation are DIP detail |
| FR-DIP-035 to 038 | FR-OBR-085, 089 (references) | TDH holds operators, routes and locations in TAM-001 and OBR; DIP's reference data maps onto them at the merge |
| FR-DIP-030 to 034 | FR-OBR-085, 086 | DIP versions profiles itself; TDH uses the M2 governance domain `import_profile` |
| FR-DIP-040 to 042 | FR-OBR-087 | DIP schema is four fields; TDH targets the full Duty, Block and Trip entities |
| FR-DIP-055 | FR-OBR-089 (source consistency) | DIP adds cross-document reconciliation |
| FR-DIP-050 to 052 | FR-OBR-089, 090 | Same statuses; DIP "Approved" maps to TDH "Accepted, transferred to draft" |
| FR-DIP-060 to 063 | FR-OBR-091, 092 | DIP exports approved records directly; TDH transfers them to a draft Schedule Version and publishes it |
| FR-DIP-070, 071 | FR-OBR-093 | Same |
| FR-DIP-080 to 089 | FR-OBR-095, 064, 063 | Same approach (openpyxl, detect and warn, reproducible). The duty-import layout becomes a TDH Export Profile / vendor layout at the merge |
| FR-DIP-090 to 093 | FR-OBR-094 | Same |
| FR-DIP-100 to 102 | NFR-OBR-005, 016 | DIP uses an encrypted local volume; TDH uses object storage (M3) |
| NFR-DIP-010 to 013 | NFR-OBR-015 to 018 | Same policy |
| NFR-DIP-020 | NFR-OBR-017 | Same |
| NFR-DIP-030 | NFR-OBR-019 | DIP targets set for Medium volume |

## Change history

| Version | Date | Change |
|---|---|---|
| 0.5 | 10 Nov 2026 | OQ-DIP-11 to 14 accepted. Batches normally hold one source format (FR-DIP-055 note). OQ-DIP-22: Trip End pre-filled from earlier approved records (FR-DIP-043). |
| 0.4 | 10 Nov 2026 | Owner answers to OQ-DIP-15 to 21: depots per operator with a default and manual mapping; target fare-matrix rule dropped and target-name checks made informational; Status Active / Approval Approve; Contract Reference from a source contract column or the route-code prefix. FR-DIP-084, 085, 087, 088 changed. |
| 0.3 | 10 Nov 2026 | Target template analysed and adopted as the preferred output (OQ-DIP-02, 09): standard record core, target and context fields (FR-DIP-040); export requirements rewritten (FR-DIP-080 to 089): sample-row removal, every field on every row, target master-data snapshot, mapping, day-type merge, export-compatibility validation, target-readiness report, output split. OQ-DIP-15 to 21 raised. No [T] items left. |
| 0.2 | 10 Nov 2026 | Owner answers to OQ-DIP-01 to 08 and the fixture analysis (DIP-FIX-001): Administrators may self-approve (flagged); responsible party; file limits 50 MB / 500 MB; section context, orientation and summary rows (FR-DIP-026 to 028); source types, reference data, operator attribution and suggestions (FR-DIP-035 to 038); core and context fields; Trip End copied only (FR-DIP-043); cross-source reconciliation (FR-DIP-055); revised business key; OQ-DIP-09 to 14. Target-dependent items now marked [T]. |
| 0.1.1 | 10 Nov 2026 | C-02: shared proxy rename to `nia_proxy` noted (owner review comment). |
| 0.1 | 10 Nov 2026 | First draft, from the master prompt, DIP-DISC-001 v0.3 decisions and OBR-001 A1 rev. 1. Items depending on the Makhado and Dolight fixtures marked [F]. |
