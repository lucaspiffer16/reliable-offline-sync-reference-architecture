# Support and Product FAQ

## Purpose

Support and product teams need answers framed in business terms, not infrastructure terminology. This document translates the architecture into stakeholder-facing language.

## FAQ

### Why can a user see "pending" even after reconnecting?

Because reconnecting is only the start of recovery. The device still needs to:

1. authenticate
2. send pending records safely
3. avoid duplicate commits
4. confirm upload validation when files are involved

Business meaning: reconnect does not guarantee immediate visibility, but it should guarantee eventual safe processing within defined objectives.

### Why do we prefer delayed records over duplicate records?

Duplicate records can create billing, operational, or support errors that are harder to unwind than short delays. The architecture therefore prioritizes replay-safe writes and conflict detection over unsafe speed.

### Why can a conflict happen if the user was offline?

Because another device or browser user may have changed the same record while the mobile user was disconnected. The backend rejects stale versions so that one user does not silently overwrite another.

### Why does an uploaded image sometimes take longer to appear than the record itself?

Because image upload and image processing are separate workflows. The record may be committed before the image is fully validated, resized, and processed.

### Can a file exist in storage but still be unavailable in the product?

Yes. A stored file is not trusted until the backend validates and confirms it. This prevents bad, mis-scoped, or malicious uploads from appearing as valid product data.

### What should support tell a customer when sync is delayed?

- their local data is still retained on the device unless the device itself failed
- the system is designed to retry safely
- support can verify whether the backend already committed the record
- if the delay exceeds service objectives, the incident should be escalated

### What should support tell a customer when the same action seems to have been retried?

The system intentionally retries in unstable networks, but retries are designed not to create duplicate committed records. Support should confirm whether the original request was committed or merely attempted.

### Why do we need tombstones for deleted records?

Because offline devices cannot infer deletion from silence. If we do not send explicit deletion state, old records can reappear or remain visible incorrectly.

### Why are there background jobs for images instead of immediate completion?

Because CPU-heavy media work would slow down customer-facing APIs and reduce scalability. Background workers keep user interactions responsive while processing continues safely.

### Why does product need to care about retry budgets and conflict rates?

Because these are not only engineering mechanics. They directly affect:

- how long users wait for visible results
- how often users must retry manually
- how often support must intervene
- whether workflows feel trustworthy

## Product Questions This Architecture Forces

- how long can a record stay pending before it becomes a customer problem?
- what should the UI show for conflict, failed upload, and retrying states?
- when should a user be asked to retry manually versus waiting for automatic recovery?
- how long must deleted records remain tombstoned for long-offline devices?

## Support Questions This Architecture Must Eventually Operationalize

- how to inspect whether a specific `request_id` committed
- how to identify whether a delay is auth-related, queue-related, or DB-related
- how to distinguish safe replay from duplicate business action
- how to handle orphaned uploads or DLQ-backed media failures
