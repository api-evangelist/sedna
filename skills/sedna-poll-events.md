---
name: Poll the event stream
description: Track platform activity by polling Sedna's event collection and resolving related resources.
api: openapi/sedna-openapi.json
operations: [getEvents, getLatestEvent, getEvent, getEventMessage, getEventJobReference, getEventCategoryTag]
scopes: [EVENT_READ, MESSAGE_READ, JOBREFERENCE_READ, CATEGORYTAG_READ]
---

# Poll the event stream

Sedna has no webhooks; consumers track activity by polling the event endpoint.

## Auth
OAuth 2.0 client-credentials token (1-hour JWT). Needs `EVENT_READ` plus the read scope of any related resource you resolve.

## Steps
1. **Establish a cursor** — call `getLatestEvent` (`GET /2019-01-01/event/latest`) once to learn the most recent event id.
2. **Page forward** — call `getEvents` (`GET /2019-01-01/event`) using the JSON:API `page[cursor]`/`page[limit]` parameters to pull new events since your last cursor.
3. **Resolve context** — for each event, follow `getEventMessage` (`GET /2019-01-01/event/{id}/message`), `getEventJobReference`, or `getEventCategoryTag` to load the related object.
4. **Persist the cursor** and repeat on an interval.

## Rules
- Pagination is cursor-based (JSON:API `page[...]`); persist the last cursor rather than re-scanning.
- Respect the 100 req/s limit; a steady poll interval avoids 429s.
- Events are read-only (`EVENT_READ`); resolving a related resource also requires that resource's read scope.
