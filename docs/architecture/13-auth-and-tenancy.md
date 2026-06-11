# Authentication, Authorization, and Tenancy

## Why This Matters

Offline-first and distributed workflows are not production-ready if identity and tenant boundaries are vague. Sync correctness without access correctness is not a serious architecture.

## Authentication Model

- Clients authenticate with short-lived access tokens.
- Mobile clients must support token refresh after long offline periods.
- All sync and upload control-plane requests require authenticated identity.

## Authorization Model

- Authorization is scoped to tenant, actor, and record ownership.
- Presigned upload authorization must validate that the caller can attach files to the target record.
- Sync responses must only include records within the caller's tenant and access scope.

## Tenant Boundaries

- PostgreSQL data must be query-scoped by tenant.
- Redis keys must be namespaced or partitioned by tenant or logical scope.
- Storage object keys must include tenant-safe prefixes.
- Audit events must preserve tenant attribution without leaking data across tenants.

## Device Identity

- Mobile requests should include a stable device identifier.
- Request-level observability should correlate user, device, tenant, and request ID.
- Device identity must never be treated as a substitute for user authentication.

## Browser Client Positioning

The browser client is intentionally preserved in the context view because it pressures the system in ways a mobile-only architecture would hide:

- concurrent edits across client types
- support and back-office visibility into canonical records
- higher likelihood of product-driven conflict scenarios

## Security Controls

- fail closed on ambiguous ownership
- treat upload confirmation as a privileged state transition
- reject idempotency replays with mismatched payload hash
- rate-limit sync and upload authorization endpoints
- audit conflict rejections, replay mismatches, and validation failures

## Open Questions For A Real Implementation

- single-tenant versus multi-tenant deployment model
- PII encryption at rest and on device
- role model for browser-office users versus field users
- device revocation and remote wipe requirements
