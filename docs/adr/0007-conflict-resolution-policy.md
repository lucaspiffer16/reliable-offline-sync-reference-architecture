# ADR 0007: Conflict Resolution Policy

## Status

Accepted

## Context

The reviewed architecture supports offline edits and delayed synchronization, which creates a risk of concurrent modification between mobile and server state.

## Decision

Use server-authoritative conflict handling with optimistic version checks. A mobile write is accepted only when it references the current remote version. Conflicts are rejected and returned to the client for retry or user review.

## Consequences

- Silent data loss is reduced compared with default last-write-wins.
- Client workflows must handle conflict responses explicitly.
- Version fields become part of the sync contract.

## Alternatives Considered

- Last-write-wins
- Automatic merge of all conflicting fields
- Manual review for every collision without version checks
