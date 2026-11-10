# Security policy — Nia Document Import Platform (DIP)

## Reporting a vulnerability

Report suspected vulnerabilities privately to the product owner (Ukwazi
Group / Gloss Data). Do not open a public issue. Include what you found,
how to reproduce it and the affected commit. You will get an
acknowledgement within two working days.

## Scope

This repository (backend, frontend, compose files, CI). DIP is internal
only (DIP-URD-001 A-01) and runs on `ukw-dev` and, later, `ukw-poc`.

## Controls in this repository

| Control | Where |
|---|---|
| Secret scanning | Gitleaks (`.gitleaks.toml`, incl. a rule for South African ID numbers), detect-secrets (`.secrets.baseline`), pre-commit hook `gitleaks protect --staged` |
| Static analysis | Semgrep (`semgrep.yml` plus registry rulesets), Bandit |
| Dependencies | pip-audit, npm audit, OWASP Dependency-Check before release (`dependency-check-suppression.xml`) |
| Secrets | Only in `.env` (never committed, never printed). `.env.example` holds key names and placeholders only |
| Client data | Never in Git. Real fixtures live outside the repo; only synthetic fixtures under `backend/tests/fixtures/synthetic/` |
| Database | Separate owner and runtime roles; the web container never holds owner credentials; append-only, hash-chained audit log (TDH M2 pattern) |
| Network | Every published port binds to 127.0.0.1; public access only through the shared reverse proxy |

## Handling personal data

DIP is designed to keep no driver identity (ADR-DIP-004, NFR-DIP-010).
No real client documents may be processed until the lawful-processing
conditions in ADR-DIP-004 §4 are met. POPIA compliance is not claimed;
review by the Information Officer is pending.

## Supported versions

Pre-release (`0.x`). Only the latest `main` is supported.
