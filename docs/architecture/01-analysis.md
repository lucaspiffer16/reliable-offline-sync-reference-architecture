# Architecture Analysis

## Main Components

- Mobile app
- Backend server
- Sync service / data synchronizer
- Asynchronous data transmission worker
- Image processor worker
- Scheduled retry worker
- Browser client

## External Systems

- Storage service
- RabbitMQ
- Redis

## Databases

- Local mobile database / SQLite
- Remote PostgreSQL

## Queues and Events

- RabbitMQ for image-processing jobs
- Scheduled local retry tasks on mobile
- Connectivity-triggered async workers for pending data

## Storage Systems

- Object storage service for uploaded images and processed variants
- Local SQLite on mobile
- Redis for short-lived validation and dedup state
- PostgreSQL as the canonical remote datastore

## Client Applications

- Mobile app
- Browser client

## Background Workers

- Mobile async worker querying unsent local orders
- Mobile data synchronizer
- Mobile scheduled worker for retry and confirmation
- Server image processor worker
- Server-side async processing path for uploads

## Synchronization Mechanisms

- Pull sync using `GET /sync?since=last_sync`
- Cursor-based windowing using backend `SELECT now()` as the upper bound
- Mobile commits `last_sync` only after all remote data is saved locally
- Push sync using a two-phase flow: UID registration followed by full payload submission
- Retry logic using Redis-backed validation and dedup checks
- Re-fetch behavior when the response is lost before the mobile commits state

## Data Ownership Boundaries

- Mobile owns offline working data, pending sync state, and local `last_sync`
- Backend/PostgreSQL owns the canonical server-side business records
- Redis owns short-lived coordination data such as UID registration and dedup markers
- Storage service owns binary media and generated derivatives
- RabbitMQ owns transient delivery of asynchronous image-processing work

## Ambiguities and Assumptions

- The diagram text says "Sync the remote data into remote database" in one place; this appears to mean syncing remote data into the mobile local database.
- The `Data Synchronizer` appears to be a mobile-side concern, though it is positioned between backend and mobile in the drawing.
- The browser client appears in the base topology but not in the detailed operational workflows.
- Conflict resolution rules are not shown explicitly; only duplicate-safe retries and repeated fetches are implied.
- Authentication, authorization, encryption, and tenancy boundaries are not shown.
- The storage service is unnamed; this documentation assumes it supports presigned uploads.
