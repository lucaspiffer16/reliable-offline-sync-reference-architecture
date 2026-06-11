# ADR 0008: Retry and Dead-Letter Policy

## Status

Accepted

## Context

The original review identified missing retry budgets, poison-message handling, and dead-letter behavior for asynchronous jobs and sync failures.

## Decision

Use bounded retries with exponential backoff and jitter for sync and processing workflows. Route exhausted or poison jobs to a dead-letter path with alerting and operator review.

## Consequences

- Transient failures are retried automatically.
- Permanent failures stop consuming capacity indefinitely.
- DLQ operations and reconciliation become required operational capabilities.

## Alternatives Considered

- Infinite retries
- Immediate manual intervention on first failure
- No DLQ, with failed jobs simply dropped
