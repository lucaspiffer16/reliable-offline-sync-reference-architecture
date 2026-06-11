# Architecture Review

## Potential Bottlenecks

- Backend may become a bottleneck during large bulk order uploads.
- Redis may become hot if every retry and dedup check hits the same key space.
- RabbitMQ consumers may lag behind upload volume for image-heavy workloads.

## Scalability Concerns

- Timestamp-based sync on `updated_at` requires proper indexing and consistent server clock handling.
- Retry loops can amplify load during prolonged network instability.
- Full order bulk POSTs may become inefficient for large payload sizes.

## Consistency Risks

- Redis and PostgreSQL can diverge if one write succeeds and the other fails.
- Re-requesting the same sync window requires strict idempotent upsert behavior on the mobile client.
- UID registration and full payload commit are split across phases, so partial completion paths need careful reconciliation.

## Security Concerns

- Presigned URLs need short expiration, constrained object keys, and content restrictions.
- UID registration and sync endpoints need authentication and replay protection.
- Storage validation is underspecified; MIME type, file size, checksum, and malware checks are not shown.

## Missing Observability

- No metrics are shown for sync lag, retry count, duplicate filtering, queue depth, or worker failure rate.
- No distributed tracing is shown across mobile sync, Redis validation, PostgreSQL persistence, and worker processing.
- No audit trail is shown for repeated sync attempts or lost-response recovery.

## Missing Failure Handling

- No dead-letter queue or poison-message handling is shown for image jobs.
- No explicit retry backoff or retry budget is defined.
- No reconciliation mechanism is shown for Redis/PostgreSQL drift.
- No explicit conflict-resolution policy is shown for concurrent edits.

## Recommendations

1. Define source-of-truth boundaries explicitly: PostgreSQL canonical, Redis ephemeral, SQLite local working copy.
2. Make all sync writes idempotent using stable record IDs and upsert semantics.
3. Formalize the two-phase UID registration and bulk upload contract, including expiry and replay behavior.
4. Add indexes on `updated_at` and relevant partition keys for sync queries.
5. Define Redis key TTLs, namespaces, and recovery rules.
6. Add dead-letter queues, worker retry policy, and alerting for RabbitMQ.
7. Add metrics for sync duration, cursor lag, retry counts, duplicate filtering, and image-processing latency.
8. Define a conflict policy such as last-write-wins, version checks, merge, or user-mediated resolution.
9. Validate uploaded files after storage write using MIME, size, checksum, and ownership checks.
10. Consider chunked or batched upload for large order payloads.

## Documentation Resolution

- Recommendation 1 is specified in `05-sync-contract.md` and ADR `0006`.
- Recommendations 2, 3, 8, and 10 are specified in `05-sync-contract.md` and ADR `0007`.
- Recommendation 5 is specified in `06-redis-and-idempotency.md`.
- Recommendations 6 and 7 are specified in `08-operations-and-observability.md` and ADRs `0008` and `0009`.
- Recommendation 9 is specified in `07-upload-security-and-validation.md`.
- Business invariants are specified in `09-domain-and-invariants.md`.
- Hidden failure modes and recovery expectations are expanded in `10-failure-matrix.md`.
- Capacity, service objectives, and scale assumptions are defined in `11-slos-and-capacity.md`.
- Contract examples are defined in `12-api-examples.md`.
- Identity and tenant boundaries are defined in `13-auth-and-tenancy.md`.
