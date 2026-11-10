**Nia Document Import Platform (DIP)**

**DIP-FIX-001 — Fixture Analysis (Phase 1, step 1.3)**

| Field | Value |
|---|---|
| Version | 0.3 — owner decisions on the target mapping recorded |
| Date | 10 November 2026 |
| Inputs | 8 source files (2 PDFs, 6 workbooks) and the target `duty-import-template_dolight.xlsx`, supplied by the owner on 10 Nov 2026 |
| Method | Structure read with openpyxl and by viewing every PDF page; OCR prototype (Tesseract 5, 300 dpi, grid-cell detection with OpenCV) on all 61 pages of the monitoring PDF, scored against the six operator workbooks as ground truth |
| Status of prototype | Throwaway feasibility test in the analysis sandbox. Not product code; its figures are indicative |

## 1. Owner answers recorded (10 Nov 2026)

| OQ | Answer |
|---|---|
| OQ-DIP-01 | The Makhado duty sheets come as a multi-page PDF covering several operators, plus an Excel workbook for each operator in it. No driver names or ID numbers. |
| OQ-DIP-02 | The Dolight files are supplied as PDF and xlsx; no driver names or IDs. **All eight supplied files are sources.** The **target** is `duty-import-template_dolight.xlsx` (to be uploaded). Target columns may be blank, as long as the agreed core fields are populated. |
| OQ-DIP-03 | Dolight is a transport operator, and it owns the template. |
| OQ-DIP-04 | Two people aren't always available. **Administrators may upload, edit and approve**, i.e. do every action. |
| OQ-DIP-05 | Makhado Bus Service is a special purpose vehicle (SPV): a group of operators contracted to the provincial department. All DIP uploaders and approvers are internal. The data belongs to the department and the SPV administration. |
| OQ-DIP-06 to 08 | Approved as recommended. OQ-DIP-08 (the duplicate business key) needs changing in the light of this analysis; see §6.3. |
| End Time | **Copied from the source only.** No derived +10-minute rule. |

## 2. Inventory

| # | File | Kind | Size | Pages / rows | Text layer | Operator(s) | Role |
|---|---|---|---|---|---|---|---|
| S1 | Makhado Bus Services Monitoring Tools 2026-2027 FY … .pdf | Scanned PDF | **22.7 MB** | 61 pages, ≈622 trip rows | None | All (SPV) | Source: department-approved timetable, organised by monitoring point |
| S2 | Year 2026 – Makhado Duty Sheet Workbook for Do Light.pdf | Scanned PDF (typed original) | 3.2 MB | 9 pages, 35 trip rows | None | Dolight | Source: operator duty sheets, one duty per page |
| S3a–f | Dolight / GNT Makhado / Mabidi / Mabirimisa / Phadziri G / Phadziri R Duties.xlsx | Workbooks | 13–21 KB | 68 / 125 / 42 / 113 / 170 / 92 rows (610 in all) | — | One each | Source: per-operator transcription of S1 |

No file contains driver names or ID numbers. **S1 does contain handwritten signatures and dates from operator and department officials, plus an official stamp, outside the timetable.** Signatures are personal information. They're never extracted, but the stored original contains them (see §6.6).

## 3. Source S1 — Monitoring Tools PDF (multi-operator)

**Layout.**
- One form throughout: "MONITORING SHEET: TIME TABLE — FOR YEAR: 2026/2027". It has a page header (Monitoring Point, Time: Morning or Afternoon, Period Week), a ruled table and a signature block.
- Each page is one section: one monitoring point × time of day × period week.
- There are 15 monitoring points: Three Mile, Elim Mall Bus Stop, Muromani, Mavhidani, Alliance Garage, Sasol Garage, Braambos, Tshililo, Thenga Cross, Safari, Makhado Bus Rank, Eltivillas Bus Rank, Tshituni (Ha-Ravele), Wylliespoort and Folovhodwe Car Wash.
- Pages hold 1 to 29 rows. A section may run over several pages (Three Mile Mon–Fri morning covers pages 1–3, rows 1–77). The row "No." restarts with each new section.

**Columns:** No., Route Code, From, To, Depart Time, Bus No, Schedule Shift no., Monitor's Shift no., Schedule Time, Monitor's Point Time, Passengers, Penalty Type, Monitor's Remarks. The last five columns, and Bus No, are blank: monitors fill them in on the day.

**Formats and quirks found:**
- Times are written `04H15`.
- The page carries `/Rotate 270`, so the table is landscape on a portrait page.
- There's a large faint "MAKHADO" logo watermark behind the table.
- The header has typos: "Period Week: **Morning** – Friday" on pages 1, 2, 4, 12 and 55, meaning Monday–Friday.
- Header labels vary: "Route" vs "Route Code"; "Passenger s" wraps; "AFTERNOON" is in capitals on page 61.
- **The operator isn't shown anywhere on the page.** It can only be worked out from the route code.

**OCR prototype results** (589 table rows detected out of ≈622; 471 matched to workbook rows):

| Field | Correct (raw OCR + cleanup) | With reference data | Note |
|---|---|---|---|
| From / To | — | **97.2% / 98.7%** | Fuzzy-matched to a list of 87 known places |
| Depart Time | **94.7%** | — | `HH H MM` pattern |
| Route Code | 65.2% | **88.5%** | Corrected where the From–To pair identifies the route |
| Schedule Time / Shift no. | 34% / 37% overall; **25/25 and 23/25** on a page where columns aligned | — | The prototype's column detection drifted across pages (14–16 vertical lines found) |
| Whole row correct | 22% | — | — |

Row detection failed on 9 sparse pages (1–8 rows), where ruled lines are short.

**Conclusion.** S1 can be read, but only with:
- (a) columns located by their **header labels**, not by counting ruled lines;
- (b) a character set constrained per column (route `VHE1_ddd`, times `ddHdd`);
- (c) **reference data** (route register with From/To, place list, operators' duty numbers) to validate and suggest corrections;
- (d) a confidence score per cell;
- (e) **mandatory review** of uncertain cells.

Expect real review effort on S1. The FR-DIP-023 "Should" (scanned PDF) is confirmed as needed for this use case.

## 4. Source S2 — Operator duty sheet PDF (Dolight)

**Layout.**
- Nine typed pages (Courier), scanned on a Canon device. They're rotated the **opposite way** to S1, so orientation must be auto-detected per page. Tesseract's orientation detection worked.
- Header: "Dolight Transport Service [PTY] LTD", which identifies the operator.
- One duty per page (168, 170, 172, 174, 176, 178, 180, 182, 192), each with a **Morning** block and an **Afternoon** block.
- Columns: Duty No, Point of Departure, Departure / **Trip start**, Arrival / **Trip end**, **Route** (as text, e.g. "Maila lagos – Makhado", with no route code), **Rev kms**, **Positioning**.
- Each block ends in a "Sub total kilometers" row. The subtotals check out (duty 168 morning: 47+47+42 = 136; positioning 5+5+42 = 52).
- No day type is printed; the "Date" field is blank. The trips match the Monday–Friday trips in S3a.

**OCR:** reading the whole page garbles it on the ruled lines, so cell-level reading is needed. The print is clean, so high accuracy is expected.

**Data-quality issues in the source itself** (DIP must surface these, not hide them):

| Duty | Issue |
|---|---|
| 178 | Afternoon trip: start 16:45, **end 15:25**, so it ends before it starts |
| 170 | Row 3: departs Masethe, but route reads "Vleifontein – Makhado" (42 km). S3a has Masethe → Makhado |
| 172 | Afternoon row 3: departs "Eltivilas 17:15", route "Eltivilas – Maila lagos". S3a has Makhado → Maila Lagos 17:15 |
| 174 | S2 has 4 weekday trips, **S3a has 5**; the 17:30 Makhado → Maila Lagos trip is missing from S2 |
| 182 | The afternoon block is empty |

## 5. Sources S3a–f — Operator workbooks

**Shape.**
- One sheet each, header in row 1, 11 columns: Rec ID, Monitoring Point, Day Type (Monday – Friday / Saturday / Sunday), Time Period (Morning / Afternoon), Route Code, From, To, Depart Time, Schedule Shift No, Schedule Time, End Time.
- Times are real Excel time values. There are no blank rows, merged cells or data validation.

**Variations between files** (each needs mapping per profile):
- Mabirimisa: column "**Monitoring Time**" instead of "Schedule Time", and sheet name "Mabirimisa".
- Phadziri G: an extra empty "**Bus No**" column (12 columns).
- Dolight: a second sheet holding a **route list**: code, description with direction, "Trunk", "Bus", "Contract". VHE1_005 is marked "Cancelled", and one entry is truncated to "Co".

**End Time:**
- In every file it's a **formula**: `=Schedule Time + TIME(0,10,0)`.
- In Phadziri G it's missing on **45 rows** (Rec IDs 304–356).
- Per the owner, DIP copies the cached value where present and leaves it blank otherwise. It never derives a value.

**Rec ID** is a single sequence 1–622 across all operators: a transcriber's numbering of S1. **12 IDs appear in no workbook**: 234, 239, 262, 264, 308, 311, 320, 326, 610, 612, 616, 617. Pages 54–57 show that 610, 612, 616 and 617 are route **VHE1_108** (Makushu ↔ Biaba, shift 117). That route isn't in any workbook. The OCR also found **VHE1_060** (Midoroni) rows, which are absent from the Phadziri G workbook.

**Route → operator.** 96 route codes in use. Each belongs to exactly one operator, in contiguous ranges:

| Operator | Routes | Duty No. pattern |
|---|---|---|
| Dolight | VHE1_001–004 | 168–192 |
| Mabidi | VHE1_006–018 | MB006–MB018 |
| Phadziri R | VHE1_019–044 | D001–D010, R01–R17 |
| Phadziri G | VHE1_045–069 | G105–G228 |
| GNT Makhado | VHE1_070–091 | 601–648 |
| Mabirimisa | VHE1_092–111 | 101–129 |

**Not in any workbook:** VHE1_005 (cancelled), 031, 035, 054, 057, 059, **060**, 064, 072, 095, 100, 105, **108**, 109, 110. Some of these do appear in S1 (060 and 108 are confirmed).

## 6. Findings that change the requirements

### 6.1 Three source profiles, one batch

DIP needs three provider-profile types:
- **P-MON**: the monitoring timetable PDF, multi-operator. Header context per page, operator worked out from the route register.
- **P-DSH**: an operator duty-sheet PDF, one duty per page. Morning and Afternoon blocks; subtotal rows used for checks, not stored as records.
- **P-WBK**: an operator duty workbook, with header-name mapping per operator.

A batch may mix them. A **reconciliation step** compares documents in the same batch.

### 6.2 Reference data becomes core

New, versioned, Administrator-maintained reference data:
- **Operators**, holding the stable operator code that maps to the TDH Operator.
- **Route register**: code, operator, From, To, description, service type (Trunk / Bus / Contract), status (active / cancelled).
- **Place list**, with aliases (e.g. ELTIVILLAS / Eltivilas).
- **Monitoring points.**

These can be seeded from the S3 workbooks and Dolight's route sheet. A row whose route isn't in the register gets processing status **Failed: unassigned operator**. It is never dropped and never guessed (e.g. VHE1_108).

### 6.3 Business key for duplicates (changes OQ-DIP-08)

The approved key (operator + duty no. + route + trip start) has **37 collisions** in the workbooks: the same trip runs Monday–Friday, Saturday and Sunday.

Proposed key: **operator + day type + duty no. + route + From + trip start**. It has zero collisions across all 610 rows.

### 6.4 Section context and normalisation

- Page-header values (monitoring point, time of day, period week → day type) are carried onto every row of the section.
- Header typos are normalised by a per-profile code list ("Morning – Friday" → Monday – Friday).
- Times like `04H15`, `04:15` and Excel time values all normalise to minutes after the start of the service day.
- Text case is normalised.

### 6.5 Cross-source consistency (new requirement)

Per operator, day type and duty, DIP compares S2 against S1 and S3:
- trip count;
- trip starts;
- From;
- route (S2's route text is matched to a code through the place list);
- trip end ≥ trip start.

All five issues in §4 must be raised as discrepancies, with links to both sources. There's **no automatic winner**; the reviewer decides (see OQ-DIP-13).

### 6.6 Personal information in originals

The S1 signature blocks are personal information. They aren't in the table, so they're never extracted. The original file (encrypted, 30-day retention, ADR-DIP-004) does contain them. Real-data processing therefore still needs the NFR-DIP-012 gate, with the responsible party recorded as **the provincial department, with the SPV administration**, and Ukwazi as processor.

### 6.7 Limits

S1 is 22.7 MB, close to the 25 MB file limit in FR-DIP-011. **Raise the limit to 50 MB per file and 500 MB per batch.** The page limit (100) is fine.

The prototype took about 7.5 minutes for 61 pages (≈7 s per page on 2 vCPU), within NFR-DIP-030's 30 s per page.

### 6.8 Fixtures in Git

S2 and S3 hold no personal information, but they're the operators' and the department's operational data. S1 contains signatures.

Proposal:
- Keep **all eight files outside Git**, in a restricted fixtures folder on `ukw-dev`.
- Commit **synthetic look-alikes** (same layouts, invented values, no signatures) as regression fixtures.
- Owner to confirm (OQ-DIP-14).

## 7. Mapping to the standard schema (sources)

| Standard field | S1 (P-MON) | S2 (P-DSH) | S3 (P-WBK) |
|---|---|---|---|
| Duty No. | Schedule Shift no. | Duty No | Schedule Shift No |
| Route | Route Code | Route text → code via place list | Route Code |
| Trip Start | Depart Time | Trip start | Depart Time |
| Trip End | **not present** | Trip end | End Time (copied; blank if missing) |
| *(kept, likely needed by target)* | Monitoring Point, Day Type, Time Period, From, To, Schedule Time, source No. | Point of Departure, Rev kms, Positioning, block | Rec ID, Monitoring Point, Day Type, Time Period, From, To, Schedule Time |

## 8. New open questions

| ID | Question | Recommendation |
|---|---|---|
| OQ-DIP-09 | Target template columns, core fields, and their mapping. | **Answered:** see §9. |
| OQ-DIP-10 | **Trip End** is a core field, but S1 doesn't have it. For trips that come only from S1, where does it come from? | Fill it from S2 or S3 for the same operator, day type, duty and trip start when either is in the batch. Otherwise mark the record **Failed: core field missing**, which a reviewer can correct. Never derive it. |
| OQ-DIP-11 | Which operator runs VHE1_060, VHE1_108 and the other routes in no workbook? Is there a seventh operator? Who maintains the route register? | Owner supplies the codes; SPV administration owns the register; a DIP Administrator maintains it. |
| OQ-DIP-12 | S2 pages don't say the day type. Assume Monday–Friday unless a page says otherwise? | Yes. Set it per document at upload, default Monday–Friday. |
| OQ-DIP-13 | When S1, S2 and S3 disagree, which value is pre-filled for review? | No automatic winner. Pre-fill from S3, then S1, for timetable fields, and from S2 for Trip End. Every difference becomes a discrepancy. |
| OQ-DIP-14 | May the supplied files be stored as regression fixtures, and where? | Outside Git on `ukw-dev`, with synthetic look-alikes in Git. |

## 9. Target — `duty-import-template_dolight.xlsx`

The owner has confirmed this is the **preferred output format for DIP** (10 Nov 2026). It's a duty import workbook for a downstream operations system. Its sample data and Lists sheet carry Calmex test routes.

**Sheets:**
- **Duties:** the import sheet. 16 columns, header in row 1, top row frozen. Data validation runs on rows 2–1001.
- **Lists:** a snapshot of the target system's master data: Depots (A), Routes (B), and each route's fare-matrix status (C: "Complete", or "Incomplete — duties on this route will be rejected"). It changes with every download from the target system.
- **Instructions:** the column guide and import rules.

| Col | Header | Target rule | Validation in template | DIP source |
|---|---|---|---|---|
| A | Duty Name | Every row; rows with the same name form one duty; **must not already exist**; ≤255 chars | — | Duty No. (core) |
| B | Depot | First row of duty; later rows blank or the same | List = Lists!A2:A3 | Operator's depot: default depot, or chosen at upload / mapped in review when the operator has several (owner) |
| C | Route | First row of duty; later rows blank = route of row above; matched ignoring case and spaces; **rejected if the fare matrix is Incomplete** | List = Lists!B2:B25 | Route code + From → To → target route name (core) |
| D | Start Time | Every row, HH:MM 24-hour | Format `hh:mm` | Trip Start (core) |
| E | End Time | Every row; **after Start Time**; **slots of one duty must not overlap on the same day** | Format `hh:mm` | Trip End (core) |
| F–L | Sun, Mon … Sat | At least one "Y" per row | List "Y" | Day type: Monday–Friday → Mon–Fri; Saturday → Sat; Sunday → Sun |
| M | Status | First row of duty | Active / Inactive | **Active** (owner) |
| N | Approval Status | First row of duty | Pending / Approve / Reject | **Approve** (owner) |
| O | Contract Route | Optional, per trip; Y / N | Y / N | Y when a Contract Reference is found (owner) |
| P | Contract Reference | Required if Contract Route = Y; ≤255 chars | — | Source column *contract* / *contract_code* / *contract_no* if present; else the route-code prefix before "_" (VHE1_001 → VHE1) (owner) |

**Writing rules for DIP:**
- The two **sample rows (D-101, D-102) must be removed.** The target imports every row left in the sheet.
- Lists and Instructions are kept unchanged, and so are the data validations. If more than 1,000 rows are written, the validation ranges are extended.
- Every field is written on every row (repeated, not left blank). That avoids the "blank Route = row above" inheritance going wrong silently.

**What the fixtures look like against the target:**

| Check | Result on the 610 workbook rows |
|---|---|
| Duty names unique across operators | Yes: 191 duties, no name shared between operators |
| End Time present | **45 rows missing** (Phadziri G) → can't be imported until filled (FR-DIP-043) |
| End after Start | All present values pass. The duty sheet (S2) has one failure (duty 178) |
| **No overlap within a duty on the same day** | **Fails for 6 slot pairs in 5 duties** (Dolight 170 ×2, GNT 607, Mabidi MB015, Phadziri G G111, Phadziri R R23), each on all 5 weekdays. The target would reject these duties. The cause is End Time, which in the workbooks is the monitoring-point time + 10 min, not the end of the trip |
| Merge identical slots across day types | 610 rows → **582** target rows |
| Route names in the target's Lists | Only **2** Makhado routes exist ("Makhado to Maila lagos", "Makhado to masethe"), and **both have an Incomplete fare matrix**. Today, **every Makhado duty would be rejected by the target**. The fixtures need **168** directional route names |
| Depots in Lists | "Operator1", "Parkhurst": test data, no Makhado operators yet |

**Consequences (as decided by the owner, 10 Nov 2026):**
- The target's row rules (end after start, no overlap within a duty on the same day, at least one day flag, consistent first-row values, contract reference when Contract Route = Y) are **export-compatibility validations** in DIP, as failures.
- **The fare-matrix rule is not applied.** It belongs to the target platform, not DIP.
- The Lists sheet is read on every template upload as an informational snapshot. Depot and route names it doesn't contain are **warnings**, listed in a target-readiness report so they can be set up in the target system.

## Change history

| Version | Date | Change |
|---|---|---|
| 0.3 | 10 Nov 2026 | Owner decisions: depot default and mapping; Status Active, Approval Approve; contract reference rule; fare-matrix rule dropped, target names informational. |
| 0.2 | 10 Nov 2026 | Target template analysed (§9): columns, rules, Lists snapshot, fixture checks (overlaps, missing routes and fare matrices). |
| 0.1 | 10 Nov 2026 | Eight source files analysed, OCR prototype scored, owner answers recorded. Target template pending. |
