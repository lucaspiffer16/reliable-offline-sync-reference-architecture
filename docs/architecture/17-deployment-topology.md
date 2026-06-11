# Deployment Topology

## Purpose

This document translates the logical architecture into a runtime view: what runs where, what scales independently, and what failure domains matter.

## Logical Runtime Components

- mobile client application
- browser client
- API service
- sync service
- background worker service
- PostgreSQL
- Redis
- RabbitMQ
- object storage
- observability stack

## Deployment Model

### Client Side

- mobile app runs on untrusted user devices
- local SQLite persists business and sync state on-device
- browser client runs in user-controlled browsers and depends on backend availability

### Server Side

- API service handles control-plane requests:
  - sync orchestration
  - UID registration
  - full payload commit
  - upload authorization and confirmation
- sync service may be deployed with the API or split if sync load becomes distinct enough
- worker service runs independently and consumes RabbitMQ jobs

### Data and Infrastructure

- PostgreSQL stores canonical business state
- Redis stores ephemeral coordination and replay markers
- RabbitMQ buffers asynchronous image-processing work
- object storage stores original and derived files
- metrics, tracing, and audit sinks collect operational evidence

## Trust Boundaries

- mobile and browser are outside the trusted backend boundary
- object storage write access is delegated through scoped presigned URLs only
- Redis is trusted for coordination, not truth
- PostgreSQL is trusted for canonical record durability

## Scaling Units

### API Nodes

- scale for request throughput, auth checks, and batch commits
- protect from large binary transfers by keeping media on direct storage paths

### Sync Nodes

- scale for read-heavy incremental sync windows
- may require isolation from write-heavy API traffic under large reconnect storms

### Worker Nodes

- scale for CPU and I/O heavy media transformations
- should be isolated from API latency-sensitive workloads

### PostgreSQL

- scale vertically first
- optimize indexes and workload shape before premature service decomposition

### Redis

- scale for low-latency coordination, not durable write amplification
- monitor memory and hot-key behavior aggressively

## Suggested Topology Evolution

### Stage 1

- API + sync in one deployable service
- one worker service
- managed PostgreSQL, Redis, RabbitMQ, and object storage

### Stage 2

- separate sync service when read-heavy traffic interferes with commit-heavy API paths
- introduce worker classes for different media or background job types

### Stage 3

- partition by tenant or workload shape if high-churn tenants dominate resources
- isolate queue consumers by priority or media class

## Failure Domains

- API deployment failure should not corrupt canonical data, but will block new sync coordination
- worker deployment failure should not block canonical order commits, but will delay media readiness
- Redis failure should degrade replay convenience, not invalidate committed truth
- PostgreSQL failure is a high-severity canonical outage
- object storage degradation blocks upload and media retrieval workflows

## Blast Radius Considerations

- reconnect storms can impact API, sync queries, Redis, and queue growth at once
- poison media jobs can consume worker capacity if DLQ handling is weak
- large tenants can create uneven load without partition-aware controls

## Deployment Questions For A Real Implementation

- single region versus multi-region
- RPO/RTO expectations for PostgreSQL and object storage
- tenant isolation requirements
- worker autoscaling inputs
- whether sync and API should remain one service or diverge
