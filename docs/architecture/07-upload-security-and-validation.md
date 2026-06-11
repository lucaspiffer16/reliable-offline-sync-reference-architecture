# Upload Security and Validation

## Presigned URL Constraints

- URLs must be short-lived.
- Each URL must be scoped to a single object key.
- Each URL must be bound to the authenticated caller's allowed record scope.
- Maximum object size must be enforced.
- Allowed content types must be restricted.

## Upload Flow Requirements

- Mobile requests upload authorization from the backend.
- Backend validates ownership and returns presigned URL(s).
- Mobile uploads directly to object storage.
- Mobile confirms uploaded object references back to the backend.
- Backend validates uploaded objects before marking them available for downstream processing.

## Validation Rules

- Verify object key belongs to the requesting actor and record.
- Verify MIME type against an allowlist.
- Verify file size is within configured limits.
- Verify checksum when available.
- Verify the object exists before queueing processing.

## Security Controls

- Do not expose generic bucket-level write access.
- Do not allow arbitrary object-key selection by the client.
- Reject expired, replayed, or mismatched upload confirmations.
- Apply malware scanning when compliance or threat model requires it.

## Post-Upload Processing Rules

- Original objects remain immutable after successful upload.
- Derived files such as thumbnails are produced by workers only.
- Processing status must be stored separately from binary storage location.

## Failure Handling

- If upload succeeds but confirmation fails, the object remains untrusted until confirmed.
- If confirmation succeeds but validation fails, the file transitions to `Failed`.
- If worker processing fails, retry according to the queue retry policy and eventually route to DLQ.
