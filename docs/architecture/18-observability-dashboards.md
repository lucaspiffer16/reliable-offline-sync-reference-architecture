# Observability Dashboards

## Purpose

Metrics only become useful when grouped into views that help operators and engineers answer real questions quickly.

## Dashboard 1: Executive Reliability Overview

### Questions Answered

- are users syncing successfully?
- are uploads and image processing within acceptable latency?
- is the platform degrading in a way that affects customers now?

### Recommended Panels

- sync success rate
- sync freshness percentile
- time from reconnect to canonical commit
- image processing latency percentile
- DLQ count
- conflict rate

### Business Interpretation

- if sync freshness rises, field teams may perceive the system as unreliable even if requests still succeed
- if conflict rate rises, product workflows may be generating user-visible friction

## Dashboard 2: Sync Pipeline Health

### Questions Answered

- are mobile reconnects converting into safe canonical writes?
- are retries helping or amplifying load?

### Recommended Panels

- requests per second for `/sync`
- batch registration success rate
- batch commit replay rate
- duplicate-prevention hit rate
- retry count by workflow
- time from first attempt to final commit

### Investigation Clues

- high replay rate with low conflict rate suggests network instability more than business contention
- rising retry count plus rising DB latency suggests reconnect storm amplification

## Dashboard 3: Data Consistency and Reconciliation

### Questions Answered

- is ephemeral coordination drifting away from canonical truth?
- are reconciliation jobs keeping up?

### Recommended Panels

- Redis/PostgreSQL drift rate
- reconciliation job duration
- missing dedup marker count
- stale request marker count
- orphaned upload count

### Investigation Clues

- increasing drift with stable DB health usually points to Redis TTL or reconciliation problems
- orphan growth with normal upload volume usually points to confirmation failures

## Dashboard 4: Media Processing Operations

### Questions Answered

- are uploads validating and being processed fast enough?
- are workers healthy and correctly isolated?

### Recommended Panels

- upload authorization rate
- upload validation failure rate by reason
- queue age
- worker failure rate
- DLQ growth by failure class
- processing latency by media type

### Investigation Clues

- queue age matters more than raw depth for customer impact
- validation failure spikes often indicate client regressions or abuse, not infrastructure failure

## Dashboard 5: Tenant and Client Release View

### Questions Answered

- is a specific tenant or client version causing trouble?
- are problems broad or isolated?

### Recommended Panels

- sync success by tenant
- conflict rate by client version
- retry exhaustion by device cohort
- upload failures by tenant
- batch size distribution by tenant

### Business Interpretation

- this dashboard is critical for incident scoping and customer communication

## Alert Design Principles

- alert on user-impacting symptoms before low-level technical noise
- alert on queue age before vanity throughput metrics
- alert on repeated retry exhaustion rather than every transient retry
- alert on drift only when it exceeds recovery tolerance

## Minimum Alert Set

- sync freshness SLO breach
- queue age breach
- DLQ growth breach
- PostgreSQL commit failure rate spike
- reconciliation drift breach
- conflict rate anomaly after release

## Trace Correlation Requirements

- correlate by tenant, user, device, `request_id`, and record UID where appropriate
- allow follow-through from mobile request to PostgreSQL commit to worker completion

## What This Repo Still Does Not Define

- exact metric names
- specific Prometheus queries
- vendor-specific dashboard configuration
