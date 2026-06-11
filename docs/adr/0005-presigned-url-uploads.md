# ADR 0005: Presigned URL Uploads

## Status

Accepted

## Context

The diagram shows the mobile client requesting presigned URLs from the backend and then sending files directly to the storage service.

## Decision

Use backend-issued presigned URLs so the mobile client uploads files directly to object storage while the backend remains responsible for authorization, metadata validation, and post-upload processing.

## Consequences

- Backend bandwidth and request time are reduced.
- Large binary transfers bypass the application server.
- The client flow becomes more complex and requires upload confirmation and validation.
- URL expiration, object-key scoping, and post-upload verification must be enforced.

## Alternatives Considered

- Upload files through the backend API
- Embed file payloads directly in business API requests
- Use a separate media gateway or proxy service
