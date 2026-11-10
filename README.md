# Nia Document Import Platform (DIP)

DIP turns transport providers' duty sheets (spreadsheets, PDFs, scans)
into traceable, reviewed, standardised records and exports them in a
destination template (first target: the Dolight duty-import workbook).
It is an interim service built to merge into Transit Data Hub (TDH) and
is retired after TDH M10b (ADR-DIP-001).

Requirements: [DIP-URD-001](docs/requirements/DIP-URD-001-requirements.md).
Architecture: [DIP-PDS-001](docs/architecture/DIP-PDS-001-architecture-spec.md).
Docs index and precedence: [docs/README.md](docs/README.md).

## Status

Phase 2 (foundation) in progress on branch `p2-foundation`. Nothing is
usable yet; this table lists only what is built and tested.

| Area | Status |
|---|---|
| Repository standards and documents | Done |
| Backend, tenancy, DB roles, audit, auth | In progress |
| Dev stack (Docker Compose) | In progress |
| Frontend shell, CI | Not started |
| Import, mapping, export, OCR | Not started (Phases 3–6) |

## Rules that matter on day one

- No real client files in Git, ever (`fixtures/README.md`).
- Never print `.env` (see CLAUDE.md).
- Every published port binds to 127.0.0.1.
