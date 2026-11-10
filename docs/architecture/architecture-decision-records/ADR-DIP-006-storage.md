# ADR-DIP-006: Encrypted local volume behind a storage interface

| Field | Value |
|---|---|
| Status | Accepted (owner, 10 Nov 2026) |
| Date | 10 November 2026 |
| Decision IDs | DIP-DISC-001 D-09 (revised) |
| Related | TDH M3 object-storage decision (MinIO or SeaweedFS, licence check pending) |

## Context

DIP is interim and runs on one host. Adding an object-storage service would mean one more container on the shared POC host. The group pattern is MinIO, but its community distribution and licence terms need checking first. TDH settles its own storage in M3.

## Decision

- All files go through one storage interface in `dip_core`: original uploads, export template files, generated exports and reports.
- The MVP implementation stores files on a dedicated Docker volume (`dip_files`), mounted only in `dip_web` and `dip_worker`.
- **Files are encrypted on write.** Each file gets its own key, and those keys are wrapped by a master key kept in a secret (`.env`, never committed). The library is `cryptography`, using AES-GCM.
- Storage names are random identifiers. No user-supplied name or path is ever used, which rules out path traversal.
- Imports never use `MEDIA_ROOT`. Uploads stream to the storage interface and are never spooled to an unencrypted temporary file.
- The master key is backed up separately from the volume. Losing it makes every stored original unreadable, by design.

## Consequences

- There's no new service. Moving to S3-compatible storage at the merge is a new implementation of the same interface.
- Key rotation and backups must be in the operations guide.
- Volume backups contain only ciphertext.
