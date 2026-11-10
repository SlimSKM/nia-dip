# ADR-DIP-004: Identity redaction at parse time; originals kept encrypted with short retention

| Field | Value |
|---|---|
| Status | Accepted (owner, 9 Nov 2026) |
| Date | 10 November 2026 |
| Decision IDs | DIP-DISC-001 D-06 |
| Related | TDH-URD-OBR-001 A1 NFR-OBR-015 to 017; OQ-OBR-18 (resolved the same way) |

## Context

Duty sheets often carry driver names, employee numbers and South African ID numbers. The master prompt requires original documents to stay traceable. POPIA §10 (minimality) and §14 (retention), and GDPR Art. 5(1)(c) and (e), require keeping as little personal information as possible, for no longer than needed. ID numbers are unique identifiers and need particular care.

## Decision

1. **Redact at parse time.** Each provider profile declares its identity columns. The pipeline also detects likely identity values in unmapped columns, including the pattern of a South African ID number (13 digits with a valid date and checksum). Identity values are replaced with a redaction marker before a raw record is stored. They never reach standard records, export records or reports. Before a batch is confirmed, the user sees a preview of what will be redacted.
2. **Keep originals encrypted, for a short time.**
   - Originals are stored encrypted at rest (ADR-DIP-006), in a restricted area.
   - Only roles that are granted it can download them, and every download is audited.
   - They are deleted after a configurable period, **default 30 days**, by a scheduled purge. The purge writes an audit event with counts only.
   - A "hold" flag with a reason, settable only by an Administrator, suspends deletion for a disputed batch.
3. **Record retention.** Raw, standard and export records, reports and audit records each have their own configurable retention setting. **No automated deletion of them is switched on** until the retention rules are documented and approved (master prompt §15).
4. **No real personal data until:**
   - a data processing agreement with Virtarix is in place;
   - the responsible party for the duty sheets is confirmed (the operator or municipality), with Ukwazi as processor, and the lawful basis is recorded;
   - the redaction tests pass on a sample document.

## Consequences

- Traceability is kept: file hash, source location and raw values survive after the original is deleted.
- DIP holds no driver identity in any lasting record.
- POPIA compliance is not claimed. This is a design measure, pending review by the Information Officer.
