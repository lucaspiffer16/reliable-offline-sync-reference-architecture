# ADR 0006: Source of Truth Boundaries

## Status

Accepted

## Context

The architecture uses PostgreSQL, Redis, mobile SQLite, and object storage. The original review identified ambiguity around data ownership and the risk of treating Redis or local state as authoritative.

## Decision

Define PostgreSQL as the canonical source of truth for remote business data, Redis as an ephemeral coordination store, mobile SQLite as a local operational store, and object storage as the source of truth for binary file content only.

## Consequences

- Consistency decisions have a clear authority chain.
- Redis outages or key loss do not redefine committed business state.
- Local mobile state can be replayed or reconciled without overriding server truth.

## Alternatives Considered

- Redis-backed authoritative sync state
- Mobile-authoritative merge model
- Mixed authority per workflow without explicit boundaries
