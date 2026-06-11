# Domain Model and Invariants

## Core Business Entities

### Order

- Represents a business transaction created or updated from mobile or web.
- Exists locally on mobile before remote commitment.
- Becomes canonical only after PostgreSQL commit.

### Order UID

- Stable identifier generated before first sync.
- Used for idempotency, deduplication, and conflict-safe retries.

### Order Version

- Monotonic version used for optimistic concurrency.
- Prevents silent overwrites across mobile and browser clients.

### Media Asset

- Binary file associated with a business record.
- Stored in object storage.
- Must remain untrusted until validated and confirmed by backend workflows.

### Sync Batch

- Logical unit of outbound mobile synchronization.
- Identified by a stable `request_id` reused across retries.

### Sync Cursor

- Server-issued timestamp that defines a bounded pull-sync window.
- Controls how inbound changes are replayed to mobile.

## Ownership Boundaries

- Mobile SQLite owns local working state and sync metadata.
- PostgreSQL owns canonical business truth.
- Redis owns ephemeral coordination and replay state.
- Object storage owns binary file content only.
- Audit and observability systems own operational evidence, not business truth.

## Non-Negotiable Invariants

1. A successful remote business commit must be recoverable from PostgreSQL alone.
2. Redis must never be the only source proving a business record was committed.
3. Mobile must never advance `last_sync` before locally persisting the full sync response.
4. The same logical order must keep the same UID across retries.
5. Replayed outbound batches must not create duplicate canonical records.
6. Upload confirmation must be scoped to both record ownership and object key.
7. A conflict must be observable to the client; it must not be silently resolved by default.
8. A failed worker must not retry forever without a terminal state.
9. A file in storage is not trusted business data until backend validation succeeds.
10. Deletions must propagate as explicit state, not as silent absence.

## Required Business Rules

- Duplicate orders are more harmful than delayed orders.
- Partial success must be visible and recoverable.
- Support teams must be able to answer: did the server commit, did the client persist, did the upload validate, did the worker finish?
- Mobile users must be able to continue capturing data while offline.

## Required Domain Extensions

The current architecture still needs explicit rules for:

- deletion and tombstone retention
- attachment ownership transfer rules
- tenant isolation and cross-tenant data visibility
- long-offline device behavior
- device replacement and local data restoration
