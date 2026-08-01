---
name: Tag and route a message to a job
description: Apply a category tag and a job reference to a message so it routes to the right team.
api: openapi/sedna-openapi.json
operations: [getCategoryTags, createCategoryTag, addCategoryTagToMessages, getJobReferences, createJobReference, addJobReferenceToMessages]
scopes: [CATEGORYTAG_READ, CATEGORYTAG_WRITE, JOBREFERENCE_READ, JOBREFERENCE_WRITE, MESSAGE_READ, MESSAGE_WRITE]
---

# Tag and route a message to a job

Classify an incoming Sedna message and attach it to a deal/job so it appears for the right team.

## Auth
OAuth 2.0 client-credentials token from `POST /oauth/token` (1-hour JWT). Needs `CATEGORYTAG_*`, `JOBREFERENCE_*` and `MESSAGE_*` scopes.

## Steps
1. **Find or create the tag** — `getCategoryTags` (`GET /2019-01-01/category-tag`); if the tag is missing, `createCategoryTag` (`POST /2019-01-01/category-tag`).
2. **Apply the tag to the message** — `addCategoryTagToMessages` (`POST /2019-01-01/category-tag/{id}/relationships/message`) with the message id(s).
3. **Find or create the job reference** — `getJobReferences` (`GET /2019-01-01/job-reference`) or `createJobReference` (`POST /2019-01-01/job-reference`).
4. **Attach the message to the job** — `addJobReferenceToMessages` (`POST /2019-01-01/job-reference/{id}/relationships/message`).

## Rules
- Relationship writes are JSON:API `add` semantics (idempotent at the relationship level — adding an existing member is a no-op).
- Prefer the V2 read endpoints (`getMessagesWithCategoryTagV2`, `getMessagesWithJobReferenceV2`) — the non-V2 variants are DEPRECATED (see lifecycle/sedna-lifecycle.yml).
- 403 means the credential lacks the scope or the underlying resource permission.
