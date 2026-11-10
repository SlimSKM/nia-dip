# Fixtures

**No real client files in this repository, ever** (DIP-TST-001 §3,
OQ-DIP-14).

- **Synthetic fixtures** (in Git): `backend/tests/fixtures/synthetic/`,
  built deterministically by a generator (from Phase 3).
- **Real fixtures** (outside Git): `/opt/nia/dip/fixtures` on `ukw-dev`,
  mode 700, owner only. Used only by the opt-in `real_fixtures` suite, and
  only after the real-data gate (NFR-DIP-012) is completed for the dev
  tenant. Never copied into the repo, never used in GitHub Actions.

`.gitignore` blocks `fixtures/real/`, and `*.pdf` / `*.xlsx` / `*.xls` /
`*.csv` everywhere except the synthetic folder.
