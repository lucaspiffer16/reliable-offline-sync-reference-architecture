# ADR 0004: Cursor-Based Synchronization

## Status

Accepted

## Context

The reviewed diagram shows pull synchronization using `GET /sync?since=last_sync` and a bounded read window based on a backend-generated timestamp cursor.

## Decision

Use cursor-based incremental synchronization where the backend captures a timestamp cursor and returns changes in the interval `(last_sync, cursor]`. The mobile app advances `last_sync` only after all returned data is persisted locally.

## Consequences

- Sync reads operate on a stable time window.
- Lost responses or partial local saves can be retried safely by reusing the previous `last_sync`.
- Client-side apply logic must be idempotent.
- Server timestamps and indexing become critical to correctness and performance.

## Alternatives Considered

- Offset-based pagination
- Full dataset refresh on each sync
- Version-counter-based sync
- Change-data-capture streaming
