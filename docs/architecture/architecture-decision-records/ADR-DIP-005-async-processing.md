# ADR-DIP-005: Celery + Redis from the PDF phase; spreadsheets synchronous until then

| Field | Value |
|---|---|
| Status | Accepted (owner, 10 Nov 2026) |
| Date | 10 November 2026 |
| Decision IDs | DIP-DISC-001 D-07, D-10 |

## Context

PDF table extraction and OCR are slow and CPU-heavy. DIP will run on the shared POC host alongside Specta, TDH and Moyo. The agreed volume (Medium) is up to 50 documents per batch, and up to 10,000 rows or 100 PDF pages per document, with daily batches.

## Decision

- **Phase 3 (Excel and CSV):** parse synchronously within the upload limits. Parsing runs in the web process, under a timeout.
- **Phase 4 (PDF and OCR):** add Celery and Redis (`dip_worker`, `dip_cache`), as TDH does.
  - Every task carries its tenant schema and sets it explicitly.
  - A worker processes at most two jobs at once.
  - Each job has CPU, memory and time limits.
  - Parsing runs in a worker container with no network access.
  - The OCR packages (Tesseract) live only in the worker image.
- Resource caps on the shared POC host: worker 2 vCPU and 2 GB; web 1 vCPU and 1 GB.

## Consequences

- Before Phase 4, only spreadsheet and CSV files can be imported.
- The Phase 4 compose file adds two services, both bound to loopback only.
