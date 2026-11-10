# ADR-DIP-002: Django + Django REST Framework on TDH's versions

| Field | Value |
|---|---|
| Status | Accepted (owner, 9 Nov 2026) |
| Date | 10 November 2026 |
| Decision IDs | DIP-DISC-001 D-03 (revised) |
| Supersedes | The master prompt's original FastAPI / SQLAlchemy / Alembic stack |

## Context

The master prompt named FastAPI, SQLAlchemy and Alembic. TDH, Specta and Fare Matrix all use Django. With FastAPI, merging DIP into TDH would mean rewriting every endpoint, model and auth piece.

## Decision

- Use Python, Django and Django REST Framework, pinned to the **same versions as TDH** (Django 5.2.x LTS), with django-tenants as in ADR-DIP-003.
- Use Django migrations. Every migration must be reversible and tested backwards (UKW-DEV-CICD-001); data migrations need a `reverse_code`.
- Validate profile and template configuration with `jsonschema` (Draft 2020-12).
- Put background work on Celery (ADR-DIP-005), not async views.
- Keep parsing, mapping, normalisation, validation, duplicate detection and template export in `dip_core`, with no Django imports and its own unit tests.
- **PDF libraries:** `pdfplumber` (with `pdfminer.six`) for text and tables, and `pypdfium2` for rendering and rotation. Tesseract runs via `pytesseract`, in the worker image only. PyMuPDF is **not** used, because its AGPL-3.0 licence conflicts with commercial use (TDH OBR-001 A1 R1).

## Consequences

- The merge mostly moves Django apps and `dip_core` into TDH.
- Auth, admin and migrations come ready-made.
- PostgreSQL is needed in every environment, including tests (ADR-DIP-003).

## Alternatives considered

FastAPI with SQLAlchemy and Alembic: fast and API-first, but it turns the merge into a rewrite.
