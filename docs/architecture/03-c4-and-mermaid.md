# C4 Model and Mermaid Diagrams

## C4 Level 1: System Context

### Actors and Systems

- Mobile app: captures data offline, syncs records, uploads images
- Browser client: accesses backend through the web interface
- Backend: coordinates sync, validation, persistence, upload authorization, and async work
- PostgreSQL: canonical data store
- Redis: ephemeral coordination and dedup store
- RabbitMQ: asynchronous job broker
- Storage service: binary object storage for media

### Boundaries

- The system of interest is the backend platform plus its persistence and async components.
- Mobile and browser are primary clients.
- Storage, queueing, and caching are supporting infrastructure dependencies.

## C4 Level 2: Container View

### Containers

- Mobile App
- Local SQLite
- Backend API
- Sync Service
- Background Workers
- PostgreSQL
- Redis
- RabbitMQ
- Storage Service

### Responsibilities

- Mobile App: local-first UX, orchestration of sync and uploads
- Local SQLite: offline persistence and sync metadata
- Backend API: request handling, validation, upload URL issuance
- Sync Service: cursor-based incremental sync
- Background Workers: image processing and asynchronous backend jobs
- PostgreSQL: source of truth for server-side records
- Redis: transient coordination for UID registration and sync dedup
- RabbitMQ: decoupling long-running tasks from request/response flows
- Storage Service: raw and processed image storage

## Mermaid Diagrams

### System Context Diagram

```mermaid
flowchart LR
    Mobile[Mobile App]
    Browser[Browser Client]
    Backend[Backend]
    Postgres[(PostgreSQL)]
    Redis[(Redis)]
    Storage[Storage Service]
    RabbitMQ[RabbitMQ]
    ImgWorker[Image Processor Worker]
    Metrics[Metrics and Alerts]
    Audit[Audit Log]

    Mobile -->|Sync API / upload coordination| Backend
    Browser -->|Web access| Backend
    Backend --> Postgres
    Backend --> Redis
    Backend --> RabbitMQ
    Backend -->|Presigned URLs / metadata| Storage
    Backend --> Metrics
    Backend --> Audit
    Mobile -->|Direct image upload via presigned URL| Storage
    RabbitMQ --> ImgWorker
    ImgWorker --> Storage
    ImgWorker --> Backend
    ImgWorker --> Metrics
```

### Container Diagram

```mermaid
flowchart LR
    subgraph Client
        Mobile[Mobile App]
        LocalDB[(Local SQLite)]
        MobileWorker[Async / Scheduled Workers]
    end

    subgraph Server
        API[API]
        Sync[Sync Service]
        Workers[Background Workers]
        DLQ[Dead Letter Queue Handler]
    end

    Queue[RabbitMQ]
    Dead[(DLQ)]
    PG[(PostgreSQL)]
    R[(Redis)]
    S[Storage Service]
    Obs[Metrics and Tracing]

    Mobile --> API
    Mobile --> Sync
    MobileWorker --> API
    Mobile --- LocalDB

    API --> PG
    API --> R
    API --> S
    API --> Queue

    Sync --> PG
    Sync --> R

    Queue --> Workers
    Queue --> Dead
    Workers --> S
    Workers --> PG
    Workers --> R
    Dead --> DLQ

    Mobile -->|Direct upload with presigned URL| S
    API --> Obs
    Sync --> Obs
    Workers --> Obs
```

### Sync Data Flow Diagram

```mermaid
flowchart TD
    A[Create or update local data on mobile] --> B[Persist in local SQLite]
    B --> C[Data Synchronizer starts]
    C --> D[Send GET /sync?since=last_sync]
    D --> E[Backend captures cursor now]
    E --> F[Query records where updated_at > last_sync and <= cursor]
    F --> G[Return changed records + cursor]
    G --> H[Mobile stores data locally]
    H --> I{All data stored successfully?}
    I -- Yes --> J[Commit last_sync = cursor]
    I -- No --> K[Do not advance last_sync]
    K --> L[Next sync re-requests same window]
    L --> M[Client upserts or deduplicates repeated records]

    B --> N[Outbound async upload path]
    N --> O[Post UID list]
    O --> P[Backend validates and registers UIDs in Redis]
    P --> Q[Post full payload]
    Q --> R[Backend persists to PostgreSQL]
    R --> S[Mark sync and dedup state in Redis]
    S --> T{Conflict or duplicate?}
    T -- Yes --> U[Filter already-synced items and retry]
    T -- No --> V[Mark as synced]
    U --> W[Apply retry backoff and audit attempt]
    W --> Q
```

### Image Upload Flow

```mermaid
sequenceDiagram
    participant M as Mobile
    participant B as Backend
    participant S as Storage
    participant Q as RabbitMQ
    participant W as Image Worker
    participant A as Audit Log

    M->>B: Request presigned URL(s)
    B-->>M: Return presigned URL(s) and upload metadata
    M->>S: Upload image(s) directly
    S-->>M: Upload success
    M->>B: Confirm uploaded file references
    B->>B: Validate metadata and file registration
    B->>A: Audit upload confirmation
    B->>Q: Enqueue image processing job
    Q->>W: Deliver processing job
    W->>S: Read original image
    W->>W: Compress, resize, generate thumbnails
    W->>S: Store processed variants
    W-->>B: Update processing status and metadata
    B-->>M: Image available or synced
```

### Sync Sequence

```mermaid
sequenceDiagram
    participant M as Mobile
    participant B as Backend
    participant D as Database
    participant R as Redis

    M->>B: GET /sync?since=last_sync
    B->>D: SELECT now() as cursor
    B->>D: SELECT * FROM orders\nWHERE updated_at > last_sync\nAND updated_at <= cursor
    D-->>B: Changed records
    B-->>M: Records + cursor
    M->>M: Persist all records locally
    alt Save succeeded
        M->>M: Update local last_sync = cursor
    else Save failed or response lost
        M->>M: Keep previous last_sync
        M->>B: Retry same sync window later
    end
    B->>R: Record ephemeral sync audit and dedup markers
```

### State Diagram

```mermaid
stateDiagram-v2
    [*] --> Pending
    Pending --> Validated: UID list accepted
    Pending --> Failed: validation failed

    Validated --> Uploading: begin full payload or image upload
    Uploading --> Uploaded: transfer complete
    Uploading --> Failed: network or storage error

    Uploaded --> Processing: backend worker starts
    Processing --> Synced: persistence or processing complete
    Processing --> Failed: worker or validation error

    Failed --> Retrying: retry scheduled
    Retrying --> Uploading: retry upload
    Retrying --> Validated: retry UID handshake
    Retrying --> Failed: retry exhausted
    Failed --> Pending: manual or scheduled requeue
    Synced --> [*]
```

### Failure Recovery Diagram

```mermaid
flowchart TD
    A[Bulk upload attempt] --> B{Request succeeded?}
    B -- Yes --> C[Commit PostgreSQL transaction]
    C --> D[Update Redis dedup state]
    D --> E[Emit metrics and audit event]
    E --> F[Done]

    B -- No --> G[Schedule retry with backoff]
    G --> H[Check Redis synced UID set]
    H --> I{Any UIDs already synced?}
    I -- Yes --> J[Filter synced records from payload]
    I -- No --> K[Retry full payload]
    J --> K
    K --> L{Retry budget exhausted?}
    L -- No --> B
    L -- Yes --> M[Move job to DLQ and alert]
```
