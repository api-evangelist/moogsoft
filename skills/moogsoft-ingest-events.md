---
name: moogsoft-ingest-events
description: Push events into Moogsoft / Dell APEX AIOps Incident Management so they deduplicate into alerts and correlate into incidents.
api: Moogsoft Events Integration API
operations:
  - postEvents
  - validateEventFilter
  - testEventWorkflow
  - getStoredInputs
---

# Ingest events into Moogsoft

Use this when you have monitoring output that should become a Moogsoft alert.

## Steps

1. **Shape the event.** The body of `POST /v1/integrations/events` (`postEvents`) takes one event or
   an array. The fields that matter are `source`, `check`, `severity`, `description`,
   `deduplication_key`, `time`, `service[]`, `tags{}` and `utc_offset`.
2. **Set the dedupe key deliberately.** Events sharing a `deduplication_key` collapse into ONE alert.
   Getting this wrong is the single most common ingestion mistake: too coarse and unrelated failures
   merge, too fine and you defeat noise reduction.
3. **Rehearse the workflow before you rely on it.** If an event workflow will transform the payload,
   call `testEventWorkflow` (`POST /v1/event-workflows/test`) or `testEventWorkflows` for bulk, and
   `validateEventWorkflow` to check the definition parses. These are read-only rehearsal endpoints.
4. **Check what actually arrived.** `getStoredInputs` (`GET /v1/event-workflows/inputs`) returns
   inputs recently seen at the front of the workflow engine — the fastest way to confirm your payload
   landed in the shape you expected.
5. **Follow the alert.** Once ingested, find it with `listAlerts` (`GET /v1/alerts`) filtered on your
   `source` or `check`, then `alertDetails` (`GET /v1/alerts/{alertId}`).

## Consequences

`postEvents` is additive and cannot be undone — an event that dedupes into the wrong alert stays
there. Validate with the test endpoints on a copy before sending production traffic.

## Ground rules for every Moogsoft call

- Base URL is `https://api.moogsoft.ai`. Authenticate with the `apiKey` header on every request.
- The key inherits the permissions of the user who created it. `GET` needs Read Only on the feature
  area; `POST`, `PATCH` and `DELETE` need Full Access. A permission failure can surface as `404`.
- Moogsoft classifies `GET` as a **safe** operation and `POST`/`PATCH`/`DELETE` as **advanced**
  operations that change platform behaviour immediately.
- **There is no idempotency key.** A retried `POST` creates a duplicate. De-duplicate before retrying.
- Every `GET` returns at most **1000 records**. Page with `start` + `limit`, or `search_after` on
  alerts and incidents.
- Errors are `{"status": "error", "message": "...", "additional": [...]}` — plain JSON, not RFC 9457.
  There is no error code; branch on the HTTP status.
- There are no rate-limit headers. Moogsoft reserves the right to throttle without warning.
