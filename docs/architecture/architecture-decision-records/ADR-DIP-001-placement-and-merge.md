# ADR-DIP-001: DIP is an interim standalone service, merged into TDH and retired after M10b

| Field | Value |
|---|---|
| Status | Accepted (owner, 9–10 Nov 2026) |
| Date | 10 November 2026 |
| Decision IDs | DIP-DISC-001 D-01, D-02; retirement timing (10 Nov) |
| Related | TDH ADR-TDH-015 (interim DIP and merge path); TDH-URD-OBR-001 Amendment A1 rev. 1, §13 |

## Context

The Makhado duty sheets need to be turned into Dolight import files now. Transit Data Hub (TDH) has the same need in its requirements (OBR-001 Amendment A1, §5.9), but TDH's import pipeline arrives only in milestones M10a (spreadsheets) and M10b (PDF and OCR), after M3, M5, M6 and M8. The owner wants a quick win now and expects to fold it into TDH later, front end included.

## Decision

1. Build the **Nia Document Import Platform (DIP)** as a standalone service. Working slug `dip`, containers `dip_<role>`, hosts `dip-dev.glossdata.co.za` and `dip-poc.glossdata.co.za`. It has its own repository, database and containers.
2. Build DIP so that it **merges into TDH**. It uses the same data model names as OBR-001 A1 §5, the TDH M5 API conventions, React feature modules for the M5 shell, and a framework-independent core package (`dip_core`).
3. Its canonical output is a duty record that maps to the TDH OBR Duty model without transformation.
4. **DIP is retired after TDH M10b**, when TDH has PDF and OCR import. Between M10a and M10b, DIP stays in service at least for PDF sources.

## Consequences

- There are two codebases until M10b, so DIP must not drift from TDH: same Django and django-tenants versions, same entity names, same API style.
- DIP data (batches, profiles, export history) is migrated into TDH at the merge, keeping ids and hashes, so source traceability survives.
- Original files move to TDH with whatever retention period they have left.
- ADR-TDH-015 is updated with the retirement point (after M10b).

## Alternatives considered

- **Build inside TDH now.** Rejected: blocked behind M3 to M8.
- **Shared library only, no service.** Rejected: there's no UI to deliver the quick win.
