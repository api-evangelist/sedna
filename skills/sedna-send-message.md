---
name: Send an email message
description: Authenticate, compose a message and send it through a Sedna shared inbox.
api: openapi/sedna-openapi.json
operations: [createMessageDraft, updateMessageDraft, sendNewMessage, sendMessage, getMessage]
scopes: [MESSAGE_WRITE, MESSAGE_READ]
---

# Send an email message

Use the Sedna API (version `2019-01-01`, base `https://{tenant}.sednanetwork.com/platform`) to compose and send email from a team inbox.

## Auth
1. Create API credentials in the tenant's API-Credential-Management page.
2. Get a token from `POST /oauth/token` using the OAuth 2.0 client-credentials flow (or use HTTP Basic with client_id/client_secret). The JWT is valid for 1 hour; request a new one after it expires. Requires the `MESSAGE_WRITE` and `MESSAGE_READ` scopes.

## Steps
1. **One-shot send** — call `sendNewMessage` (`POST /2019-01-01/message/send`) with the recipients, subject, body and sending team to create and send in a single call.
2. **Draft-then-send** (when you need review) — call `createMessageDraft` (`POST /2019-01-01/message`), optionally revise with `updateMessageDraft` (`PATCH /2019-01-01/message/{id}`), then `sendMessage` (`POST /2019-01-01/message/{id}/send`) or `updateAndSendMessagePatch` (`PATCH /2019-01-01/message/{id}/send`).
3. **Confirm** — `getMessage` (`GET /2019-01-01/message/{id}`) to read back the sent message.

## Rules
- Requests/responses follow JSON:API conventions; attach relationships (team, category-tag, job-reference) via the `/relationships/` sub-resources.
- Rate limit: 100 requests/second per key or IP — back off on HTTP 429.
- There is no idempotency key; do not blindly retry `sendNewMessage`/`sendMessage` on a network error — reconcile with `getMessage` first.
- Errors use HTTP status codes with a JSON `Error` body (401 = bad credentials, 403 = missing scope/permission). See errors/sedna-problem-types.yml.
