# ADR-DIP-003: django-tenants from day one, one tenant

| Field | Value |
|---|---|
| Status | Accepted (owner, 10 Nov 2026) |
| Date | 10 November 2026 |
| Decision IDs | DIP-DISC-001 D-05 |
| Related | TDH ADR-TDH-001 (schema-per-tenant, per-tenant users) |

## Context

The master prompt says not to build full commercial multi-tenancy for the MVP. TDH uses one schema per tenant, with per-tenant user accounts. Adding tenancy to DIP later would mean moving the data between schemas at the merge.

## Decision

- Use django-tenants on the same version as TDH, with **one tenant** in the MVP.
- Copy the TDH pattern: users, sessions and roles live in the tenant schema, and the public schema holds platform staff only. Tenant accounts can never be superuser or staff.
- There is no tenant self-service, billing or second tenant in the MVP.

## Consequences

- DIP tenant data lifts straight into a TDH tenant schema at the merge.
- Tests run against PostgreSQL, not SQLite.
- Background tasks must always set the tenant schema explicitly (ADR-DIP-005).
