# Failure Matrix

## Purpose

This document identifies the failure cases that define whether the architecture is credible. Happy-path diagrams are not enough for production systems.

## Critical Failure Cases

| Scenario | Immediate Risk | Expected System Behavior | Recovery Mechanism | Remaining Risk |
| --- | --- | --- | --- | --- |
| Mobile loses network before sync request reaches backend | User sees delayed sync | Local data remains pending | Scheduled retry with backoff | Growing local queue |
| Backend returns sync response but mobile crashes before local commit | Same data may be fetched again | `last_sync` is not advanced | Replay-safe local upsert on next sync | Repeated heavy windows |
| Backend commits PostgreSQL but Redis write fails | Dedup marker missing | Canonical commit remains valid | Reconciliation job rebuilds Redis | Temporary extra duplicate checks |
| Redis records request marker but PostgreSQL commit fails | False impression of acceptance | Request must not be treated as durable | Expire or clear Redis marker | Short replay ambiguity window |
| Mobile posts same batch twice after timeout | Duplicate commercial action | Server detects replay by `request_id` and UID | Idempotent response or duplicate-safe no-op | Payload mismatch abuse attempt |
| Upload reaches storage but confirmation never arrives | Orphaned object | Object remains untrusted | Cleanup sweep for stale unconfirmed uploads | Storage cost growth |
| Confirmation arrives but uploaded file fails validation | Corrupt or malicious asset | File marked `Failed` | Reject processing and audit event | User re-upload required |
| Worker processes image but crashes before metadata update | Duplicate processing risk | Retry must be idempotent | Worker replay based on immutable input and output checks | Extra compute cost |
| Queue backlog spikes after reconnect storm | High sync latency | Queue depth and backlog alerts trigger | Autoscaling or traffic shaping | Degraded user wait time |
| PostgreSQL is healthy but `updated_at` query slows under large tenants | Pull sync latency rises | Queries still correct but slower | Indexing, batching, partitioning, query review | Tenant hot spots |
| Multiple devices edit same order offline | Conflicting business truth | Server rejects stale version writes | Conflict response with current canonical version | Manual resolution burden |
| Delete occurs on web while mobile is offline | Ghost record risk | Mobile receives tombstone on next sync | Tombstone retention and idempotent delete apply | Long-offline retention gap |
| Auth token expires during long offline period | Upload or sync blocked after reconnect | Client re-auth required before sync | Session renewal flow | Delayed eventual consistency |
| Redis outage during UID registration | Coordination degraded | Fallback policy applies or request fails closed | PostgreSQL uniqueness checks or retry later | Higher latency or lower availability |
| DLQ grows silently | Permanent failure hidden from operators | Alerts and dashboards surface it | Operator review and replay/runbook | Support backlog |

## Hidden Failure Modes Still Not Fully Solved

- local SQLite corruption
- mobile app version upgrade with incompatible local schema
- extreme long-offline devices returning after data-retention windows
- clock or timezone confusion leaking into client logic
- large batches where one poisoned record repeatedly blocks progress
- tenant-level abuse through repeated oversized uploads

## Operator Questions This Architecture Must Answer

- Was the record committed remotely?
- Was the response delivered to the client?
- Did the client persist the data locally?
- Was the upload validated?
- Did the image worker finish or dead-letter?
- Was the conflict user-caused or replay-caused?

## Follow-Up Design Work

- define schema migration policy for local SQLite
- define support runbooks for orphaned uploads and replay disputes
