# API Contract Examples

## Purpose

These examples turn the architecture into something engineers can review, challenge, and eventually implement.

## Pull Sync Request

```http
GET /sync?since=2026-04-15T10:00:00Z
Authorization: Bearer <token>
X-Device-Id: device_123
X-Tenant-Id: tenant_acme
```

## Pull Sync Response

```json
{
  "cursor": "2026-04-15T10:05:00Z",
  "records": [
    {
      "uid": "ord_01J123ABC",
      "version": 7,
      "status": "confirmed",
      "updated_at": "2026-04-15T10:03:10Z"
    }
  ],
  "tombstones": [
    {
      "uid": "ord_01JOLD999",
      "deleted_at": "2026-04-15T10:02:00Z"
    }
  ]
}
```

## UID Registration Request

```http
POST /sync/batches/register
Authorization: Bearer <token>
Content-Type: application/json
Idempotency-Key: batch_req_001
```

```json
{
  "request_id": "batch_req_001",
  "device_id": "device_123",
  "uids": ["ord_01J123ABC", "ord_01J123ABD"]
}
```

## UID Registration Response

```json
{
  "request_id": "batch_req_001",
  "results": [
    {"uid": "ord_01J123ABC", "status": "accepted"},
    {"uid": "ord_01J123ABD", "status": "already_synced"}
  ],
  "expires_at": "2026-04-16T10:00:00Z"
}
```

## Full Payload Submission

```http
POST /sync/batches/commit
Authorization: Bearer <token>
Content-Type: application/json
Idempotency-Key: batch_req_001
```

```json
{
  "request_id": "batch_req_001",
  "records": [
    {
      "uid": "ord_01J123ABC",
      "version": 3,
      "payload": {
        "customer_id": "cus_100",
        "total": 149.90,
        "currency": "EUR"
      }
    }
  ]
}
```

## Successful Commit Response

```json
{
  "request_id": "batch_req_001",
  "status": "committed",
  "committed_uids": ["ord_01J123ABC"],
  "already_synced_uids": []
}
```

## Replay-Safe Commit Response

```json
{
  "request_id": "batch_req_001",
  "status": "replayed",
  "committed_uids": ["ord_01J123ABC"],
  "already_synced_uids": []
}
```

## Conflict Response

```http
409 Conflict
```

```json
{
  "code": "VERSION_CONFLICT",
  "uid": "ord_01J123ABC",
  "expected_version": 4,
  "received_version": 3,
  "current_record": {
    "uid": "ord_01J123ABC",
    "version": 4,
    "status": "approved"
  }
}
```

## Presigned Upload Authorization Request

```http
POST /uploads/authorize
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "record_uid": "ord_01J123ABC",
  "files": [
    {
      "name": "photo.jpg",
      "content_type": "image/jpeg",
      "size_bytes": 5242880,
      "checksum": "sha256:abcd1234"
    }
  ]
}
```

## Presigned Upload Authorization Response

```json
{
  "uploads": [
    {
      "file_key": "tenant_acme/orders/ord_01J123ABC/original/photo.jpg",
      "upload_url": "https://storage.example.com/...",
      "expires_at": "2026-04-15T10:10:00Z",
      "max_size_bytes": 5242880,
      "content_type": "image/jpeg"
    }
  ]
}
```

## Upload Confirmation Request

```http
POST /uploads/confirm
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "record_uid": "ord_01J123ABC",
  "file_key": "tenant_acme/orders/ord_01J123ABC/original/photo.jpg",
  "checksum": "sha256:abcd1234"
}
```

## Validation Failure Response

```http
422 Unprocessable Entity
```

```json
{
  "code": "UPLOAD_VALIDATION_FAILED",
  "reason": "MIME_TYPE_NOT_ALLOWED",
  "file_key": "tenant_acme/orders/ord_01J123ABC/original/photo.jpg"
}
```

## Notes

- These examples are illustrative and intentionally compact.
- A real implementation should define pagination, auth claims, error taxonomies, and schema versioning explicitly.
