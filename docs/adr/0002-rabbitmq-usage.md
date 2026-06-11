# ADR 0002: RabbitMQ Usage for Asynchronous Processing

## Status

Accepted

## Context

The architecture includes image upload and post-processing steps such as compression, resize, and thumbnail generation. These tasks should not block request/response latency in the backend API.

## Decision

Use RabbitMQ to queue asynchronous image-processing work and decouple job production from worker execution.

## Consequences

- API latency remains low while long-running work is executed asynchronously.
- Worker throughput can be scaled independently.
- Queue durability, retries, dead-letter handling, and poison-message controls become operational requirements.

## Alternatives Considered

- Synchronous image processing in the API
- In-process background threads
- Database-backed job table
- Periodic polling jobs
