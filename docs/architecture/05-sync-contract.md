# Sync Contract and Source of Truth

## Source of Truth Boundaries

- PostgreSQL is the canonical source of truth for remote business records.
- Redis is an ephemeral coordination layer only.
- Mobile SQLite is the local operational store used for offline behavior and sync bookkeeping.
- Object storage is the source of truth for binary file content, but not for business authorization state.

## Record Identity

- Every order and uploadable entity must have a stable global identifier.
- Mobile-generated records must use collision-resistant identifiers before first sync.
- Retries must reuse the same identifiers and must never generate replacement IDs for the same logical record.

## Inbound Pull Sync Contract

- Endpoint shape: `GET /sync?since=<last_sync>`
- Backend behavior:
  - capture a server-side `cursor`
  - query records in the interval `(last_sync, cursor]`
  - return data, tombstones, and the cursor in the same response
- Mobile behavior:
  - persist all returned records using idempotent upserts
  - apply returned tombstones idempotently
  - update local `last_sync` only after all writes complete successfully
  - keep the previous `last_sync` when writes fail or when the response is lost

## Pagination and Windowing Rules

- Large sync windows may be paginated, but every page must remain bound to the same server-issued cursor.
- The client must not advance `last_sync` until the final page in the cursor window is fully persisted.
- If a page fails, the client must replay the same cursor window safely.

## Deletion and Tombstone Rules

- Deletions must be represented explicitly as tombstones during sync.
- Tombstones must include at least stable UID and deletion timestamp.
- Tombstones must be retained long enough to serve long-offline clients.
- Silent omission is not a valid delete propagation strategy.

## Outbound Push Sync Contract

### Phase 1: UID Registration

- Mobile sends a list of stable record IDs and a request identifier.
- Backend validates the caller, records the request in Redis, and returns one of:
  - `accepted`: UID is eligible for full upload
  - `already_synced`: UID is already committed remotely
  - `rejected`: UID is invalid or unauthorized

### Phase 2: Full Payload Submission

- Mobile sends only UIDs that are `accepted`.
- The request must include the same request identifier used in phase 1.
- Backend must treat replayed requests as idempotent.
- Backend writes PostgreSQL first, then records dedup state in Redis.

## Idempotency Rules

- Repeating a pull sync for the same `(last_sync, cursor]` window must not corrupt local state.
- Repeating UID registration must return the same effective outcome while the request remains valid.
- Repeating full payload submission must not create duplicate records.
- Image upload confirmation must be idempotent on file key plus owning record ID.

## Conflict Resolution Policy

- Default policy: server-authoritative with optimistic version checks.
- If the mobile submits an older version than the current remote version:
  - reject the write as a conflict
  - return the current remote version
  - keep the local record in `Failed` or `Needs Review` state
- If the mobile submits the expected current version:
  - accept and increment the server version
- Last-write-wins is explicitly rejected as the default because it hides data loss.

## Retry and Replay Rules

- Retry attempts must use exponential backoff with jitter.
- The same request identifier must be reused across retries of the same logical upload batch.
- A batch must have a finite retry budget.
- After retry exhaustion, the batch transitions to a dead-letter or operator-review path.
- Retrying a partially successful batch must allow already-synced records to be filtered out safely.

## Payload Sizing and Batching

- Full order uploads must support batching.
- A batch must have a bounded record count and maximum payload size.
- Failed batches must be splittable into smaller retry batches.

## Required Database Indexes

- Index `updated_at` for sync window queries.
- Index any tenant, account, or device partition key that scopes sync.
- Enforce unique constraints on stable business identifiers.
