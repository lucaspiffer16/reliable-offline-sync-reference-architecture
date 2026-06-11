# Operations, Reliability, and Observability

## Retry Policy

- Use exponential backoff with jitter for sync retries.
- Define separate retry budgets for:
  - UID registration
  - full payload submission
  - image processing jobs
- Persist retry count and next-attempt time in local mobile state for client-originated work.

## Dead-Letter Handling

- RabbitMQ-backed image jobs must have a dead-letter queue.
- Jobs move to DLQ after retry exhaustion or poison-message detection.
- DLQ events must trigger alerting and operator review.

## Reconciliation Jobs

- Run a periodic job that rebuilds Redis dedup state from PostgreSQL.
- Run a periodic job that identifies upload confirmations without completed processing.
- Run a periodic job that reports repeated mobile retry exhaustion.

## Metrics

- Sync duration
- Cursor lag
- Number of records per sync batch
- Retry count by workflow
- Duplicate-filter rate
- Queue depth
- Worker failure rate
- DLQ count
- Image-processing latency

## Tracing

- Trace pull sync requests from mobile to PostgreSQL query execution.
- Trace outbound upload batches from UID registration through final PostgreSQL commit.
- Trace image uploads from presigned URL issuance through worker completion.

## Audit Events

- Lost-response recovery
- Replay request accepted
- Replay request rejected due to payload mismatch
- Conflict rejection
- Upload validation rejection
- DLQ placement

## Alerting

- Alert on sync lag above agreed thresholds.
- Alert on queue depth growth and worker backlog.
- Alert on spikes in retry exhaustion or conflict rates.
- Alert on Redis reconciliation drift beyond tolerance.

## Capacity and Scaling Controls

- Cap maximum records per bulk upload batch.
- Cap maximum image uploads per request.
- Scale workers based on queue depth and processing latency.
- Review `updated_at` query plans regularly as data volume grows.
