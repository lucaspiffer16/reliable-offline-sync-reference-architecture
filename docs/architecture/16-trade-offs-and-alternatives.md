# Trade-Offs and Alternatives

## Purpose

This document explains why the architecture is shaped this way, what was rejected, and where the chosen design is weaker than the alternatives.

## Offline-First Versus Online-Only

### Chosen

- offline-first with local SQLite and background sync

### Why

- field workflows cannot depend on continuous connectivity
- user productivity is more important than architectural simplicity

### Cost

- reconciliation complexity
- local schema migration burden
- conflict and retry handling become first-class requirements

### Rejected Alternative

- online-only architecture with minimal local cache

### Why It Was Rejected

- simpler backend behavior would come at the cost of business interruption in weak-network environments

## Cursor-Based Pull Sync Versus Event Stream Replication

### Chosen

- timestamp cursor windows using `(last_sync, cursor]`

### Why

- simpler to explain and implement
- adequate for many field-data systems before scale becomes extreme

### Cost

- weaker semantics around deletes and long windows
- potential scaling pain for very large high-churn tenants

### Rejected Alternative

- log-based or event-stream synchronization

### Why It Was Rejected

- stronger semantics come with much higher implementation and operational complexity for a reference architecture at this stage

## PostgreSQL-First Truth Versus Redis-Heavy Coordination Truth

### Chosen

- PostgreSQL canonical truth, Redis ephemeral coordination

### Why

- business correctness must survive cache loss
- recovery is easier when durable truth is singular

### Cost

- replay and dedup paths may be slightly slower than a cache-first design
- reconciliation jobs are still needed

### Rejected Alternative

- treating Redis as the effective authority for sync state

### Why It Was Rejected

- it would make correctness depend on TTLs, cache durability, and operational luck

## Two-Phase UID Registration Versus Direct Full Upload

### Chosen

- UID registration followed by full payload commit

### Why

- reduces wasted transfer for already-synced records
- creates an explicit surface for replay detection and duplicate filtering

### Cost

- more protocol complexity
- more edge cases between phase 1 and phase 2

### Rejected Alternative

- single-shot full payload upload every time

### Why It Was Rejected

- simpler transport, but higher duplicate risk, more wasted bandwidth, and weaker control over recovery behavior

## Presigned Uploads Versus Proxy Upload Through Backend

### Chosen

- direct client upload to storage via presigned URLs

### Why

- reduces backend bandwidth and request latency
- scales better for media-heavy workflows

### Cost

- more complex client flow
- stronger validation and cleanup requirements

### Rejected Alternative

- backend proxying binary uploads

### Why It Was Rejected

- easier trust boundary, but poor scalability and infrastructure efficiency

## Async Image Processing Versus Synchronous Processing

### Chosen

- queue-backed asynchronous processing

### Why

- protects user-facing latency from CPU-heavy work
- allows independent worker scaling

### Cost

- requires queues, DLQ, retries, and worker observability

### Rejected Alternative

- process images inline during API request

### Why It Was Rejected

- poor latency profile and low resilience under burst workloads

## Optimistic Concurrency Versus Last-Write-Wins

### Chosen

- server-authoritative version checks

### Why

- avoids silent overwrite of user actions
- turns conflicts into explicit product events

### Cost

- more client complexity
- operational burden when conflict rates spike

### Rejected Alternative

- last-write-wins

### Why It Was Rejected

- simpler, but unsafe for business records where silent overwrite creates trust loss

## RabbitMQ Versus No Queue

### Chosen

- queue-backed async jobs for image processing

### Why

- separates user flows from long-running work

### Cost

- more infrastructure and more operational burden

### Rejected Alternative

- no queue, only in-process async tasks

### Why It Was Rejected

- less reliable, less scalable, and harder to recover after crashes

## Where This Architecture Will Break First

- large high-churn tenants with timestamp-window sync
- conflict-heavy collaborative editing
- weak operational maturity around Redis reconciliation
- media-heavy tenants without worker autoscaling and queue observability
- long-offline devices returning after tombstone retention windows

## What A Future Version Might Change

- move from pure timestamp sync toward versioned or event-based replication for large tenants
- replace some coordination logic with workflow engines such as Temporal
- add stronger multi-region or tenant-isolation strategies
- formalize outbox or CDC patterns for backend-originated integrations
