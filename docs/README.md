# DIP documentation

## Precedence

Where documents differ: **URD (requirements) > PDS (architecture) >
phase prompts (scope details)**. Conflicts are flagged, never resolved
silently. ADRs record decisions; the URD and PDS cite them. Once code
exists, `docs/` in Git is authoritative for implemented behaviour, and
only built-and-tested behaviour is documented as working.

## Index

| Area | Document | What it is |
|---|---|---|
| Discovery | [DIP-DISC-001](discovery/DIP-DISC-001-phase0-discovery-report.md) | Phase 0 discovery report and decisions D-01 to D-12 |
| Requirements | [DIP-URD-001](requirements/DIP-URD-001-requirements.md) | Business, functional and non-functional requirements (governs) |
| Requirements | [DIP-FIX-001](requirements/DIP-FIX-001-fixture-analysis.md) | Analysis of the supplied Makhado sources and the Dolight target |
| Architecture | [DIP-PDS-001](architecture/DIP-PDS-001-architecture-spec.md) | Architecture, data model and API specification |
| Decisions | [ADR-DIP-001](architecture/architecture-decision-records/ADR-DIP-001-placement-and-merge.md) | Interim standalone service, merged into TDH, retired after M10b |
| Decisions | [ADR-DIP-002](architecture/architecture-decision-records/ADR-DIP-002-django-drf.md) | Django + DRF on TDH's versions; no PyMuPDF |
| Decisions | [ADR-DIP-003](architecture/architecture-decision-records/ADR-DIP-003-tenancy.md) | django-tenants from day one, one tenant |
| Decisions | [ADR-DIP-004](architecture/architecture-decision-records/ADR-DIP-004-personal-data-and-retention.md) | Identity redaction at parse time; encrypted originals, short retention |
| Decisions | [ADR-DIP-005](architecture/architecture-decision-records/ADR-DIP-005-async-processing.md) | Celery + Redis from the PDF phase |
| Decisions | [ADR-DIP-006](architecture/architecture-decision-records/ADR-DIP-006-storage.md) | Encrypted local volume behind a storage interface |
| UX | [DIP-UX-001](user-guide/DIP-UX-001-wireframes.md) | Low-fidelity wireframes |
| Testing | [DIP-TST-001](testing/DIP-TST-001-test-strategy.md) | Test strategy |
| Prompts | [prompts/](prompts/) | Phase prompts for Claude Code |

Phase 2 adds `api/` (OpenAPI), `developer-guide/` and `BUILD-LOG.md`.
