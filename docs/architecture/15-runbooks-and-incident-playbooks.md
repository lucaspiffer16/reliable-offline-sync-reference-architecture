# Runbooks and Incident Playbooks

## Purpose

Architecture without operator guidance is incomplete. These runbooks define what the team should do when the most important failure modes occur.

## Operating Principles

- PostgreSQL is the canonical source of truth for committed business records.
- Redis is coordination state and may be rebuilt.
- Storage objects are untrusted until validated.
- Never mark a workflow resolved only because a cache key exists.
- Prefer bounded containment over infinite automatic retry.

## Incident 1: PostgreSQL Commit Succeeded, Redis Marker Missing

### Symptoms

- user claims a record was already sent
- retry path does not recognize the UID as already synced
- canonical record exists in PostgreSQL

### Immediate Checks

1. confirm canonical row exists in PostgreSQL
2. confirm dedup marker is absent or expired in Redis
3. inspect audit trail for the original `request_id`

### Response

1. treat PostgreSQL as authoritative
2. rebuild Redis dedup marker from canonical state
3. replay response to client as already committed if needed
4. check whether reconciliation job is lagging or failing

### Escalate When

- the same gap appears repeatedly across tenants
- reconciliation jobs are failing
- duplicate submissions are being accepted despite canonical presence

## Incident 2: Upload Reached Storage but Was Never Confirmed

### Symptoms

- object exists in storage
- no validated media record exists in backend state
- user cannot see processed image

### Immediate Checks

1. verify object key exists
2. verify confirmation API call did not complete or was rejected
3. verify whether object has exceeded unconfirmed retention threshold

### Response

1. keep object untrusted
2. if within retention window, allow client to retry confirmation safely
3. if outside retention window, quarantine or delete object according to policy
4. emit audit event for orphaned upload handling

### Escalate When

- orphan rate spikes
- storage costs are growing abnormally
- confirmation failures correlate with auth or client version issues

## Incident 3: DLQ Growth for Image Workers

### Symptoms

- DLQ count increasing
- image processing latency breaching objective
- repeated failures on same message class

### Immediate Checks

1. inspect DLQ payload sample
2. classify failure as transient, poison payload, dependency outage, or code defect
3. check worker deployment changes and storage health

### Response

1. stop automatic replay for poison messages
2. replay only validated transient failures
3. patch validation or worker logic if defect is systemic
4. communicate user-facing delay if image processing SLA is impacted

### Escalate When

- failure class is unknown
- worker crash loop continues after rollback or config correction
- customer workflows are blocked by missing images

## Incident 4: Reconnect Storm Causes Sync Latency Spike

### Symptoms

- sudden increase in sync request rate
- rising queue age or API latency
- many devices transitioning from offline to online together

### Immediate Checks

1. check request rate, queue age, and DB latency
2. identify whether the storm is regional, tenant-specific, or global
3. inspect retry backoff behavior from active clients

### Response

1. protect PostgreSQL with rate limits or batch caps if required
2. increase worker or API capacity if safe
3. enforce smaller batch retries for overloaded clients
4. prioritize canonical writes over non-critical background work

### Escalate When

- sync freshness SLO is breached for priority tenants
- PostgreSQL saturation risks broader outage
- retries continue amplifying instead of settling

## Incident 5: Conflict Rate Spikes After Mobile Release

### Symptoms

- sudden increase in `VERSION_CONFLICT`
- support reports users seeing failed or review-required records
- new client version recently deployed

### Immediate Checks

1. compare conflict rate by client version
2. inspect whether version field handling changed in mobile logic
3. verify browser or office workflows are not mass-editing the same records

### Response

1. treat conflicts as product-visible incidents, not background noise
2. roll back or hotfix broken client behavior if version handling regressed
3. provide support guidance for manual resolution path
4. audit whether conflict states are visible enough in UI

### Escalate When

- conflict rate exceeds expected baseline for a tenant or release
- support cannot resolve the workflow operationally
- users are blocked from submitting critical business records

## Routine Operational Jobs

### Reconciliation Job

- rebuild missing Redis dedup markers from PostgreSQL
- remove stale request markers
- report drift rate as an observable metric

### Orphan Cleanup Job

- detect unconfirmed uploads older than allowed threshold
- quarantine or delete according to retention policy
- audit cleanup actions

### Local Retry Audit

- identify clients or tenants with repeated retry exhaustion
- flag possible schema, auth, or product workflow issues

## Minimum Dashboard Views

- sync freshness and success rate
- time from reconnect to remote commit
- queue age and DLQ growth
- duplicate-prevention and replay outcomes
- upload validation failure rate
- conflict rate by client version and tenant

## What This Repo Still Lacks

- exact operational thresholds per tenant tier
- pager severity mapping
- disaster recovery runbooks for PostgreSQL and storage outages
