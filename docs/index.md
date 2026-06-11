# Documentation Index

## Start Here

If you want the fastest path through the repository:

1. `../README.md`
2. `architecture/09-domain-and-invariants.md`
3. `architecture/03-c4-and-mermaid.md`
4. `architecture/05-sync-contract.md`
5. `architecture/10-failure-matrix.md`
6. `architecture/16-trade-offs-and-alternatives.md`
7. `adr/`

## Architecture Documents

- `architecture/01-analysis.md`: extracted components, assumptions, and ambiguities from the original diagram
- `architecture/02-concerns.md`: separation of static architecture, data flow, workflows, state, and operations
- `architecture/03-c4-and-mermaid.md`: context, container, sequence, state, and failure-recovery diagrams
- `architecture/04-review.md`: production risks, weaknesses, and review recommendations
- `architecture/05-sync-contract.md`: sync windows, replay rules, batching, tombstones, and conflicts
- `architecture/06-redis-and-idempotency.md`: Redis role, key design, TTLs, and reconciliation behavior
- `architecture/07-upload-security-and-validation.md`: upload authorization, trust boundaries, and validation rules
- `architecture/08-operations-and-observability.md`: retries, DLQ, reconciliation, metrics, traces, and alerts
- `architecture/09-domain-and-invariants.md`: business entities and non-negotiable correctness rules
- `architecture/10-failure-matrix.md`: critical failure cases and expected recovery behavior
- `architecture/11-slos-and-capacity.md`: service objectives, scale assumptions, and what breaks first
- `architecture/12-api-examples.md`: illustrative API contracts for sync, conflicts, and uploads
- `architecture/13-auth-and-tenancy.md`: authentication, authorization, and tenant scoping
- `architecture/14-evolution-roadmap.md`: staged evolution from reference docs to executable slice
- `architecture/15-runbooks-and-incident-playbooks.md`: operator actions for common high-impact incidents
- `architecture/16-trade-offs-and-alternatives.md`: explicit rationale for chosen and rejected approaches
- `architecture/17-deployment-topology.md`: runtime topology, scaling boundaries, and blast-radius thinking
- `architecture/18-observability-dashboards.md`: dashboard slices and how to interpret failures operationally
- `architecture/19-support-and-product-faq.md`: stakeholder-facing answers for product, support, and customer impact

## ADRs

- `adr/0001-offline-first-architecture.md`
- `adr/0002-rabbitmq-usage.md`
- `adr/0003-redis-usage.md`
- `adr/0004-cursor-based-synchronization.md`
- `adr/0005-presigned-url-uploads.md`
- `adr/0006-source-of-truth-boundaries.md`
- `adr/0007-conflict-resolution-policy.md`
- `adr/0008-retry-and-dead-letter-policy.md`
- `adr/0009-observability-and-auditability.md`
