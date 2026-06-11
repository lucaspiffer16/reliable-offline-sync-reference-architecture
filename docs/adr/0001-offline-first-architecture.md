# ADR 0001: Offline-First Architecture

## Status

Accepted

## Context

The mobile application must continue capturing and editing business data during intermittent or absent network connectivity. The reviewed diagram shows local storage, background workers, and retry behavior centered on mobile resiliency.

## Decision

Adopt an offline-first mobile architecture using a local SQLite database as the operational store. Synchronization with the backend happens asynchronously through pull sync for inbound changes and staged push sync for outbound data.

## Consequences

- The mobile experience remains functional while offline.
- Sync state such as `last_sync`, pending uploads, and retry tasks must be persisted locally.
- The system must support idempotent retries, replay-safe requests, and duplicate handling.
- Conflict resolution becomes an explicit architectural concern.

## Alternatives Considered

- Online-only mobile application
- Read-only offline cache with server-authoritative writes
- Manual export/import or operator-assisted synchronization
