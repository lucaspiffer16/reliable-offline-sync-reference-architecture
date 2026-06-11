# Redis and Idempotency Specification

## Redis Responsibilities

- Store ephemeral UID registration results.
- Store request idempotency markers.
- Store short-lived synced UID lookup sets for retry filtering.
- Store short-lived sync audit markers if needed for operational debugging.

## Redis Non-Responsibilities

- Redis must not be treated as a durable system of record.
- Redis must not be the only place where a successful business commit is represented.
- Redis loss must degrade performance and coordination, not destroy canonical business state.

## Key Namespaces

- `sync:req:{request_id}`: request metadata and replay status
- `sync:uid:{uid}`: UID registration status
- `sync:synced:{scope}:{uid}`: short-lived synced marker
- `upload:file:{record_id}:{file_key}`: upload confirmation marker

## TTL Policy

- Request markers: 24 hours
- UID registration markers: 24 hours
- Synced UID retry markers: 7 days
- Upload confirmation markers: 24 hours

TTL values are operational defaults and may be adjusted based on retry windows and support workflows.

## Idempotency Semantics

- `request_id` identifies one logical sync batch.
- If `sync:req:{request_id}` already exists for the same caller and payload hash, return the stored result.
- If the same `request_id` appears with a different payload hash, reject the request.
- Redis keys must be written atomically per logical operation where possible.

## Reconciliation Rules

- PostgreSQL success plus Redis failure:
  - treat PostgreSQL as authoritative
  - rebuild Redis markers asynchronously
- Redis success plus PostgreSQL failure:
  - clear or expire the Redis marker
  - never report the record as durable until PostgreSQL commits
- Periodic reconciliation job:
  - rebuild missing synced markers from PostgreSQL
  - remove stale or orphaned request markers

## Redis Outage Behavior

- Pull sync should continue if Redis is unavailable.
- UID registration may degrade to PostgreSQL-backed checks or fail closed depending on security requirements.
- Full payload submission must not bypass canonical uniqueness checks in PostgreSQL.

## Hot Key Avoidance

- Partition keys by tenant, account, or logical scope.
- Avoid single global sets for all synced UIDs.
- Prefer bounded sets or hashed partitions when retry volume grows.
