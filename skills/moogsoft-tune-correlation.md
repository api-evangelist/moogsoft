---
name: moogsoft-tune-correlation
description: Read and change the Moogsoft correlation definitions and groups that decide how alerts become incidents — the highest-consequence configuration surface in the platform.
api: Moogsoft Config API
operations:
  - getAllCorrelationDefinitions
  - getDefinition
  - createCorrelationDefinition
  - modifyCorrelationDefinition
  - deleteCorrelationDefinition
  - getAllCorrelationGroups
  - createCorrelationGroup
  - modifyCorrelationGroup
  - deleteCorrelationGroup
---

# Tune Moogsoft alert correlation

Moogsoft's own documentation singles this out: if you PATCH or DELETE a correlation definition, the
change immediately affects how the instance correlates alerts and creates incidents. Treat every
write here as a production change.

## Steps

1. **Read the current state and keep it.** `getAllCorrelationDefinitions`
   (`GET /v2/correlation/definitions`) and `getAllCorrelationGroups`
   (`GET /v2/correlation/groups`). Save the full payloads — there is no version history and no undo.
2. **Read one.** `getDefinition` (`GET /v2/correlation/definitions/{identifier}`) by name, or
   `getDefinition2` (`GET /v2/correlation/definitions/id/{identifier}`) by id. The same by-name and
   by-id pairing exists for groups.
3. **Add rather than edit where you can.** `createCorrelationDefinition`
   (`POST /v2/correlation/definitions`) introduces new behaviour without disturbing what already
   works.
4. **Edit deliberately.** `modifyCorrelationDefinition`
   (`PATCH /v2/correlation/definitions/{identifier}`). The effect is immediate on the next alert.
5. **Deleting is final.** `deleteCorrelationDefinition` and `deleteCorrelationGroup` have no restore
   endpoint and no retention window. The only recovery is re-creating from the payload you saved in
   step 1, and the identifier will change.

## Watch the blast radius

After any change, sample `listIncidents` and `getSituationRoomAlerts` over the following window to
confirm incidents are still forming the way responders expect. Correlation changes do not raise
errors when they are wrong — they quietly change what gets paged.

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
