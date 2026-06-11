# ADR 0003: Redis Usage for Ephemeral Coordination

## Status

Accepted

## Context

The diagram uses Redis for short-lived records related to unique identifiers from mobile orders and as part of retry and deduplication checks.

## Decision

Use Redis as an ephemeral coordination store for UID registration, deduplication markers, and transient sync validation state.

## Consequences

- UID checks and retry coordination are fast and operationally lightweight.
- The system avoids overloading PostgreSQL with transient coordination data.
- Expiry rules and reconciliation paths are required to avoid Redis/PostgreSQL drift.

## Alternatives Considered

- PostgreSQL-only coordination tables
- In-memory application cache
- Queue-only coordination without a dedicated ephemeral store
