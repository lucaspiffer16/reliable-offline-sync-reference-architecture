# Reliable Offline Sync Reference Architecture

A production-oriented reference architecture for offline-first mobile data capture, replay-safe synchronization, resilient media uploads, and observable backend workflows.

This repository is designed to demonstrate architecture judgment expected from backend and mobile engineers working on:

- distributed systems
- offline-first applications
- reliability engineering
- sync and conflict handling
- production operations

## Problem

Field applications do not fail in clean data centers. They fail when users are offline, reconnect all at once, upload large files over unstable networks, and assume the system will never lose or duplicate their work.

This repository models a system where:

- a mobile app must keep working without internet access
- business records must sync safely after reconnect
- image uploads must not overload the backend
- retries must not create duplicate records
- operators must be able to explain what happened when sync goes wrong

## Business Context

This architecture fits businesses where delayed or duplicated records create operational pain:

- field sales
- inspections
- logistics
- healthcare mobility
- maintenance operations
- proof-of-work or media-heavy workflows

The design assumes that:

- delayed synchronization is acceptable within defined limits
- silent data loss is not acceptable
- duplicate commercial records are often worse than delayed records
- mobile devices are unreliable nodes in a distributed system

## What This Repository Demonstrates

- offline-first mobile architecture with local SQLite
- cursor-based pull synchronization
- two-phase outbound sync with UID registration and idempotent replay handling
- PostgreSQL as canonical truth and Redis as ephemeral coordination
- presigned URL uploads with post-upload validation
- RabbitMQ-backed asynchronous image processing
- retry budgets, DLQ, reconciliation jobs, and observability requirements
- ADR-driven architecture documentation

## System Guarantees

- mobile does not advance `last_sync` before local persistence succeeds
- duplicate sync retries do not create duplicate remote records
- direct uploads do not bypass backend authorization or validation
- Redis loss degrades coordination, not canonical truth
- failed async jobs do not retry forever without visibility
- conflicts are explicit, not silently overwritten

## What This Repository Does Not Claim

- it is not a full implementation
- it is not a benchmarked production deployment
- it does not claim exactly-once delivery
- it does not claim conflict-free collaboration across arbitrary data types

This is a reference architecture package intended to show engineering thinking, trade-offs, and operational realism.

## Architecture At A Glance

- Mobile app persists local work in SQLite.
- Backend exposes sync and upload orchestration endpoints.
- PostgreSQL stores canonical business records.
- Redis stores ephemeral request, dedup, and coordination state.
- Storage service receives direct client uploads through presigned URLs.
- RabbitMQ and workers process images asynchronously.
- Metrics, tracing, audit events, and reconciliation jobs provide operational visibility.

## Repository Structure

- `docs/index.md`: guided documentation entry point
- `docs/architecture/01-analysis.md`: extracted architecture inventory and assumptions
- `docs/architecture/02-concerns.md`: static architecture, data flow, workflows, state, and operations
- `docs/architecture/03-c4-and-mermaid.md`: C4-style documentation and Mermaid diagrams
- `docs/architecture/04-review.md`: production review, risks, and recommendations
- `docs/architecture/05-sync-contract.md`: source-of-truth boundaries and sync contract
- `docs/architecture/06-redis-and-idempotency.md`: Redis keying, TTLs, reconciliation, and replay behavior
- `docs/architecture/07-upload-security-and-validation.md`: presigned upload and media-validation rules
- `docs/architecture/08-operations-and-observability.md`: retries, DLQ, metrics, tracing, and alerts
- `docs/architecture/09-domain-and-invariants.md`: business entities, ownership, and non-negotiable rules
- `docs/architecture/10-failure-matrix.md`: critical failure modes and recovery behavior
- `docs/architecture/11-slos-and-capacity.md`: service objectives, limits, and scale assumptions
- `docs/architecture/12-api-examples.md`: contract examples for sync, upload, conflict, and replay
- `docs/architecture/13-auth-and-tenancy.md`: identity, authorization, and tenant boundaries
- `docs/architecture/14-evolution-roadmap.md`: how this design should evolve from reference docs to implementation
- `docs/architecture/15-runbooks-and-incident-playbooks.md`: operator actions for the most important failure cases
- `docs/architecture/16-trade-offs-and-alternatives.md`: rejected options, cost of choices, and where this design breaks down
- `docs/architecture/17-deployment-topology.md`: logical deployment model, scaling units, and trust boundaries
- `docs/architecture/18-observability-dashboards.md`: dashboard design, operational slices, and alert interpretation
- `docs/architecture/19-support-and-product-faq.md`: business-facing answers for support, product, and stakeholder alignment
- `docs/adr/`: architecture decision records

## Why The Browser Client Exists

The browser client remains in the context diagrams because many real systems expose the same canonical records to office or support users through a web interface. It is not the focus of this repository, but it matters because:

- it introduces multi-client consistency pressure
- it makes conflict handling more realistic
- it prevents the mobile design from being artificially isolated

## Recommended Reading Order

1. `docs/architecture/09-domain-and-invariants.md`
2. `docs/architecture/03-c4-and-mermaid.md`
3. `docs/architecture/05-sync-contract.md`
4. `docs/architecture/10-failure-matrix.md`
5. `docs/architecture/08-operations-and-observability.md`
6. `docs/architecture/16-trade-offs-and-alternatives.md`
7. `docs/adr/`

## Hiring Signal Intent

This repository is meant to signal:

- backend and mobile systems thinking
- pragmatic distributed systems judgment
- failure-oriented design
- operational maturity beyond happy-path system design
- ability to communicate architecture to engineers, managers, and recruiters

## Next Step For This Repository

The next meaningful evolution is a minimal executable reference slice:

- sync endpoint
- idempotent bulk upload flow
- Redis-backed request markers
- worker retry and DLQ behavior
- test cases for replay, conflict, and duplicate prevention

## Source

- reviewed source diagram: `/home/lucaspiffer/Downloads/Simple Diagram.drawio`
