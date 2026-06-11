# Separated Concerns

## 1. Static Architecture

### Components

- Mobile app
- Local SQLite database
- Backend API
- Sync service
- Async upload workers
- Image processor worker
- Scheduled retry worker

### Infrastructure

- PostgreSQL
- Redis
- RabbitMQ
- Object storage service

### Dependencies

- Mobile depends on local SQLite, backend API, and presigned upload URLs
- Backend depends on PostgreSQL, Redis, RabbitMQ, and storage
- Image processing depends on RabbitMQ and storage
- Sync logic depends on PostgreSQL and local mobile sync metadata

## 2. Data Flow

- Mobile creates and updates orders locally in SQLite
- Mobile pulls server updates using `last_sync`
- Backend computes a bounded cursor and returns records updated in `(last_sync, cursor]`
- Backend returns records and tombstones for the same bounded window
- Mobile persists remote records locally, applies tombstones, then updates `last_sync`
- Mobile pushes unsent local records by first posting a UID list
- Backend stores and validates those UIDs in Redis and responds
- Mobile posts the full order payload
- Backend persists the canonical data in PostgreSQL
- Backend updates Redis cache/dedup state
- Mobile retries failed uploads and filters already-synced items
- Mobile requests presigned URLs and uploads images directly to storage
- Backend and workers validate and process uploaded images asynchronously

## 3. Business Workflows

- Offline order capture on mobile
- Background upload of new local orders
- Incremental pull sync of remote updates back to mobile
- Direct image upload followed by async image processing
- Retry and resume behavior after intermittent connectivity
- User feedback when there is no local data to send

## 4. State Management

### Sync State

- `last_sync`
- Server cursor captured at sync time
- Local pending sync queue

### Upload State

- Unassigned local orders
- UID registered
- Full payload pending
- Fully synced

### Retry State

- Retry scheduled
- Redis validation pending
- Already synced
- Retryable failure
- Terminal failure

### Image State

- Pending
- Validated
- Uploading
- Uploaded
- Processing
- Synced
- Failed
- Retrying

## 5. Operational Concerns

- Redis is used for short-lived UID validation and deduplication
- RabbitMQ is used for decoupled image-processing work
- Scheduled retry tasks recover from lost responses and failed uploads
- Cursor-based sync windows reduce missed updates during in-flight reads
- Re-fetch behavior relies on idempotent local upserts
- The design requires tombstone retention, rate limits, and tenant-aware scoping
