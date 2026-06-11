# SLOs, Capacity, and Scaling Assumptions

## Why This Document Exists

Architectures that do not define acceptable failure and latency boundaries usually fail through ambiguity, not technology.

## Proposed Service Objectives

### Sync Freshness

- Objective: 95% of successfully connected mobile sync attempts complete within 10 seconds for standard batches.
- Business consequence: field users can trust that reconnecting does not create long visible delays.

### Eventual Convergence

- Objective: 99% of valid mobile records reach canonical PostgreSQL within 5 minutes of stable network recovery.
- Business consequence: support and operations can reason about when remote data should be visible.

### Duplicate Prevention

- Objective: duplicate remote commits caused by transport retries should remain below 0.01% of sync batches.
- Business consequence: commercial and operational records stay trustworthy.

### Image Processing Latency

- Objective: 95% of valid uploaded images are processed within 2 minutes.
- Business consequence: proof-of-work and image-backed workflows remain usable.

### DLQ Visibility

- Objective: 100% of DLQ placements generate an alert or operational event within 5 minutes.
- Business consequence: permanent failures do not remain invisible.

## Non-Goals

- zero data loss under arbitrary device failure
- exactly-once delivery
- strong consistency while offline
- sub-second convergence for large reconnect storms

## Capacity Assumptions

- standard sync batch: up to 100 records
- large sync batch: up to 500 records
- maximum full payload size: defined per tenant and endpoint
- maximum images per request: bounded to protect queue and storage fan-out
- reconnect storms are expected after temporary regional network recovery

## Capacity Controls

- enforce batch limits at API boundary
- split retry batches when failures repeat
- cap concurrent upload authorizations per device
- cap worker concurrency per node for CPU-heavy image transforms
- protect PostgreSQL with tenant-aware query and write limits

## Scaling Strategy

### PostgreSQL

- index sync query paths
- isolate hot tenant patterns early
- consider partitioning when `updated_at` scans become a dominant cost

### Redis

- avoid global dedup sets
- shard coordination keys by scope
- monitor memory and eviction behavior as first-class risks

### RabbitMQ and Workers

- scale workers independently from API nodes
- alert on queue age, not just queue depth
- prioritize DLQ visibility over raw throughput vanity metrics

### Mobile

- prevent local retry storms with persisted backoff
- expose pending and failed state clearly to users

## SLI Candidates

- sync success rate
- sync duration
- time from reconnect to remote commit
- duplicate-prevention hit rate
- conflict rate
- upload validation failure rate
- image processing latency
- queue age and DLQ count
- Redis reconciliation drift rate

## What Breaks First

- poorly indexed sync queries under high-churn tenants
- retry amplification during reconnect storms
- Redis hot keys during large duplicate-filter workloads
- worker backlog during media-heavy spikes
- operator blind spots when audit events are incomplete
