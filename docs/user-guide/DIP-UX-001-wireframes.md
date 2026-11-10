**Nia Document Import Platform (DIP)**

**DIP-UX-001 — UI Wireframes (Phase 1, step 1.7)**

| Field | Value |
|---|---|
| Version | 0.1 — low-fidelity, for owner review |
| Date | 10 November 2026 |
| Basis | DIP-URD-001 v0.5, DIP-PDS-001 v0.1.1 |
| Conventions | TDH navigation rules: breadcrumbs on every page, a back-to-parent link, safe cancel (unsaved changes prompt), validated `next` redirects. Brand: Ukwazi Group / Nia (Baobab palette). Desktop first, usable on a tablet; the review table scrolls sideways on narrow screens and stays keyboard-operable. |

Legend: `[Button]`, `(•)` radio, `[x]` checkbox, `▾` dropdown, `⚠` warning, `✖` error, `ⓘ` info, `★` self-approved flag.

## Site map

```
Dashboard
├── Batches ── New batch (upload) ── Batch ── Document ── Mapping
│                                       ├── Review (Records · Duplicates · Reconciliation · Target readiness)
│                                       └── Approve
├── Exports ── New export ── Export file
├── Reports
├── Reference data (Operators & depots · Routes · Places · Monitoring points · Seed from workbook)
├── Configuration (Profiles · Export templates)        [Administrator]
└── Admin (Users · Retention · Audit log · Real-data gate) [Administrator]
```

## W1 — Dashboard

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ NIA · DIP                 Dashboard   Batches   Exports   Reports   ⚙  [User▾]│
├──────────────────────────────────────────────────────────────────────────────┤
│ Home                                                                         │
│ ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐   │
│ │Batches (7d)│ │Records     │ │ ✖ Failed   │ │Pending     │ │Exports (7d)│   │
│ │     4      │ │   1,212    │ │    51      │ │review  138 │ │     6      │   │
│ └────────────┘ └────────────┘ └────────────┘ └────────────┘ └────────────┘   │
│ Recent batches                                          [New batch]          │
│ ┌──────────┬──────────────────┬────────────┬───────┬──────┬────────┬───────┐ │
│ │ Batch    │ Source           │ Status     │ Rows  │ ✖    │Pending │By     │ │
│ │ B-0012   │ Monitoring PDF   │ In review  │  622  │  45  │  130   │ S.P.  │ │
│ │ B-0011   │ Workbook (6)     │ Processed  │  610  │   6  │    8   │ S.P.  │ │
│ └──────────┴──────────────────┴────────────┴───────┴──────┴────────┴───────┘ │
│ Recent exports                         Records per week ▁▃▅▇▅  Failure rate 4%│
└──────────────────────────────────────────────────────────────────────────────┘
```

## W2 — New batch (upload)

```
Home › Batches › New batch                                         [Cancel]
┌───────────────────────────── 1 Files ────────────────────────────────────────┐
│  ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐                              │
│    Drop PDF, XLSX, XLS or CSV files here, or [Browse]                        │
│    ≤ 50 MB per file · ≤ 500 MB per batch · ≤ 100 pages                       │
│  └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘                              │
│  Makhado Monitoring Tools 2026-27.pdf  22.7 MB  PDF (scanned) ✔             │
│  ⚠ Same file already uploaded in B-0009 (12 Nov) [View]   [Upload anyway]    │
│  macro-test.xlsm   ✖ Rejected: macro-enabled workbooks are not accepted      │
├───────────────────────────── 2 Settings ─────────────────────────────────────┤
│ Source type   (•) Monitoring timetable  ( ) Duty sheet  ( ) Workbook         │
│ Profile       [Makhado monitoring timetable v3 ▾]   (suggested by layout)    │
│ Operator      [— from route register (multi-operator) — ▾]                   │
│ Day type      [Monday–Friday ▾]   (duty sheets only; default Mon–Fri)         │
│ Depot         Default depot per operator ⓘ  [Choose per operator…]           │
│ Export target [Duty import — Dolight template v2 ▾]                          │
├───────────────────────────── 3 Identity redaction preview ───────────────────┤
│ ⓘ No identity columns found. Signature blocks are outside the table and     │
│   are not extracted. (Shows redacted columns and counts when found.)         │
│                                                     [Back] [Create & process]│
└──────────────────────────────────────────────────────────────────────────────┘
```

The real-data gate: while it's off, an Administrator sees a banner, "Real client documents can't be processed until the real-data gate is completed", with a link to W12.

## W3 — Batch detail

```
Home › Batches › B-0012                         [Reprocess] [Review] [Approve]
Status: In review · Monitoring PDF · profile v3 · created 12 Nov 10:02 by S.P.
┌ Documents ─────────────────────────────────────────────────────────────────┐
│ Makhado Monitoring Tools…pdf  61 pages  622 rows  ✖45 ⚠130  Original: kept │
│ until 12 Dec [Download ⓘ audited] [Hold]                                   │
├ Units (all selected by default) ───────────────────── [Select all] [None] ─┤
│ [x] p1–3  THREE MILE · Morning · Mon–Fri      77 rows  conf 91%            │
│ [x] p4–5  THREE MILE · Afternoon · Mon–Fri    49 rows  conf 88%            │
│ [ ] p13   ELIM MALL · Morning · Saturday   skipped: "duplicate of p11" ✎   │
│ …                                                                          │
├ Totals ────────────────────────────────────────────────────────────────────┤
│ Units 61 = 60 processed + 1 skipped · Rows 622 = raw records 622 ✔ reconciled│
└────────────────────────────────────────────────────────────────────────────┘
```

## W4 — Mapping (per document)

```
Home › Batches › B-0011 › Mabirimisa Duties.xlsx › Mapping           [Cancel]
Layout fingerprint matches profile "Operator workbook v2" ✔
┌──────────────────────┬─────────────────────┬────────────┬──────────────────┐
│ Source column        │ Sample              │ Maps to    │ Confidence       │
├──────────────────────┼─────────────────────┼────────────┼──────────────────┤
│ Schedule Shift No    │ 110, 101            │ Duty No ▾  │ ✔ saved mapping  │
│ Route Code           │ VHE1_102            │ Route code▾│ ✔                │
│ Depart Time          │ 04:00               │ Trip start▾│ ✔                │
│ Monitoring Time      │ 06:50               │ Schedule ▾ │ ⚠ alias          │
│ End Time  (formula)  │ 07:00               │ Trip end ▾ │ ✔ cached value   │
│ Bus No               │ (empty)             │ — keep, unmapped ▾ │          │
└──────────────────────┴─────────────────────┴────────────┴──────────────────┘
Core fields: Duty No ✔ Route ✔ Trip start ✔ Trip end ✔
                         [Use for this batch]  [Save to profile as draft v3]
```

## W5 — Review (records)

```
Home › Batches › B-0012 › Review
Tabs: [Records 622] [Duplicates 3] [Reconciliation 0] [Target readiness 168]
Filters: Status [Failed ▾] Rule [any ▾] Operator [any ▾] Duty [   ] Confidence < [80%]
┌──┬───────┬────────┬──────────┬───────────────┬──────┬──────┬─────────┬──────┬────────┐
│☐ │Op     │Duty    │Route     │From → To      │Start │End   │Days     │Proc. │Review  │
├──┼───────┼────────┼──────────┼───────────────┼──────┼──────┼─────────┼──────┼────────┤
│☐ │DOLIGHT│170     │VHE1_002  │Masethe→Makhado│05:30 │07:00 H│M T W T F│✖ overlap│Pending│
│☐ │PHADZ_G│G111    │VHE1_053  │Ravele→Makhado │05:50 │  —   │M T W T F│✖ end missing│Pending│
│☐ │ ?     │117     │VHE1_108⚠ │Makushu→Biaba  │05:15 │  —   │M T W T F│✖ unassigned│Pending│
└──┴───────┴────────┴──────────┴───────────────┴──────┴──────┴─────────┴──────┴────────┘
  H = Trip End from history (earlier approved record)   [Approve selected] [Reject…]

Record drawer (click a row) ───────────────────────────────────────────────────
│ Source: Monitoring PDF p2, line 16 · THREE MILE · Morning · Mon–Fri  [View page]│
│ ┌ page image crop with the row highlighted ┐                                 │
│ Field       Original     Normalised   Conf   Source                           │
│ Route code  VHE1 004     VHE1_004     62%  ⚠ From–To suggests VHE1_001 [Use] │
│ Trip end    —            07:00        —    History: B-0011 rec #41 [Open]     │
│ Results  ✖ DUTY_SLOT_OVERLAP with 04:30–05:55 (same duty, Mon–Fri) [Show]     │
│ Edit: Trip end [ 05:25 ]  Reason [ corrected from duty sheet   ] [Save edit]   │
│ History: edited by … · approved by … ★ self-approved                          │
```

**Duplicates tab:** one row per candidate, showing its level (file, row, in-document, across batches, export), the records involved, and a decision of Keep both / Skip / Replace, with a reason.

**Reconciliation tab:** per operator, day type and duty, the check that failed (count, start, From, route, end before start, missing in source), with links to both sources.

**Target readiness tab:** depot and route names not in the template's latest Lists snapshot, with the affected duties. It's informational.

## W6 — Approve

```
Home › Batches › B-0012 › Approve
Eligible: 571 records (no errors; warnings overridden or none)  Not eligible: 51
Scope (•) All eligible  ( ) Selected (0)  ( ) Filter…
Warnings to override: 12  [Review overrides…]  (each needs a reason)
ⓘ You imported this batch. As Administrator you may approve; it will be marked ★.
                                                        [Approve 571 records]
```

A Reviewer who imported the batch, or edited a record, sees the button disabled, with "You can't approve your own work".

## W7 — New export

```
Home › Exports › New export
Template   [Duty import — Dolight template v2 ▾]  Lists snapshot 9 Nov 2026
Records    (•) Approved, not yet exported in batch [B-0012 ▾]  ( ) Filter…
Options    [x] Merge identical slots across day types   Split (•) per operator ( ) one file
Defaults   Status [Active ▾]  Approval Status [Approve ▾]
Contract   Reference from source column, else route-code prefix (VHE1) ⓘ
Checks     ✔ End after start  ✖ 6 overlapping slots (5 duties) [Fix in review]
           ✔ Day flags  ✔ Contract references  ⓘ 166 route names not in target Lists
                                               [Generate] (disabled while ✖)
```

## W8 — Export file and history

```
Home › Exports › E-0031
DOLIGHT-duty-import-2026-11-12.xlsx · 60 rows from 68 records · template v2
SHA-256 4f1c…9a2 · generated 12 Nov 11:40 by S.P. · reproducible ✔   [Download]
Rows ▸ row 2 ← records #3, #115, #186 (Mon–Fri, Sat, Sun merged)
History: E-0029 (superseded by E-0031, not overwritten) · E-0027 …
```

## W9 — Reports

```
Home › Reports
Type   [Batch processing ▾] (Failures · Duplicates · Awaiting review · Rejected ·
        Approved · Export history · Audit history · Summary by date range)
Scope  Batch [B-0012 ▾]   or  From [   ] To [   ]
Format (•) XLSX ( ) CSV ( ) PDF                                  [Generate]
Recent reports: …  [Download]
```

## W10 — Reference data

```
Home › Reference data › Routes          [Seed from workbook…] [Add route]
Filter: Operator [any ▾] Status [active ▾]  Search [VHE1_0     ]
┌──────────┬───────────┬──────────────┬──────────────┬───────────────────────┬────────┐
│ Code     │ Operator  │ From         │ To           │ Target name           │ Status │
│ VHE1_001 │ DOLIGHT   │ Makhado      │ Maila Lagos  │ Makhado to Maila lagos│ active │
│ VHE1_005 │ DOLIGHT   │ Makhado      │ Masethe      │ —                     │cancelled│
│ VHE1_108 │ — ⚠       │ Makushu      │ Biaba        │ —                     │ draft  │
└──────────┴───────────┴──────────────┴──────────────┴───────────────────────┴────────┘
Seed preview: 6 operators · 96 routes (192 directions) · 87 places — [Commit as draft]
Operators & depots: DOLIGHT → depots: Dolight Main (default) [+ depot]
```

## W11 — Configuration (Administrator)

**Profiles:** a list showing version and status (draft / approved). The editor uses form sections (units, header, columns, aliases, identity columns, normalisation lists, rules and severities, confidence threshold), with a JSON view. Validation errors name the failing JSON Schema rule. **[Approve version]** makes a version live.

**Export templates:** [Upload template] → a summary of sheets and columns, the sample rows that will be removed, the unsupported parts list (must be accepted), the column map from standard fields, and the Lists snapshot. **[Approve version]**.

## W12 — Admin

```
Users: name · roles (Admin, Import operator, Reviewer, Read-only) · MFA ✔ · last sign-in
Retention: Originals 30 days (automated ✔) · Raw/Standard/Export/Reports 730 days
           (recorded, not automated) · Audit 1,825 days (recorded)
Audit log: filters actor / action / date · [Verify chain ✔ OK 12 Nov 08:00]
Real-data gate: Responsible party [Dept of Transport & Community Safety + SPV admin]
  Lawful basis [ … ]  Virtarix DPA ref [ … ]  Redaction test passed on [date]
  Status: ✖ incomplete → real client documents blocked          [Save]
```

## Change history

| Version | Date | Change |
|---|---|---|
| 0.1 | 10 Nov 2026 | First low-fidelity wireframes for W1–W12. |
