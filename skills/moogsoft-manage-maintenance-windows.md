---
name: moogsoft-manage-maintenance-windows
description: Schedule, inspect, cancel and skip Moogsoft maintenance windows so planned work does not page anyone.
api: Moogsoft Config API
operations:
  - createMaintenanceWindow
  - getMaintenanceWindows
  - getMaintenanceWindow
  - updateMaintenanceWindow
  - getMaintenanceOccurrences
  - getPendingMaintenanceOccurrences
  - getRunningMaintenanceOccurrences
  - cancelOccurrence
  - skipOccurrence
  - deleteMaintenanceWindow
---

# Manage Moogsoft maintenance windows

This is the one Moogsoft surface with a real, documented reversal path — use it.

## Steps

1. **Schedule.** `createMaintenanceWindow` (`POST /v1/maintenance/windows`). A window has a filter
   selecting what it suppresses, a schedule, and a timezone drawn from the published valid-timezone
   list.
2. **Confirm it exists.** `getMaintenanceWindows` (`GET /v1/maintenance/windows`) then
   `getMaintenanceWindow` (`GET /v1/maintenance/windows/{id}`).
3. **Inspect occurrences.** A window generates occurrences. `getMaintenanceOccurrences`
   (`GET /v1/maintenance/windows/{id}/occurrences`), `getPendingMaintenanceOccurrences`,
   `getRunningMaintenanceOccurrences`, `getMaintenanceExpiredOccurrences`, and
   `getMaintenanceWindowStatus` (`GET /v1/maintenance/windows/{id}/status`).
4. **Change your mind — this is reversible.** `cancelOccurrence`
   (`PUT /v1/maintenance/occurrences/{id}/cancelled`) cancels an UPCOMING occurrence.
   `skipOccurrence` (`PUT /v1/maintenance/occurrences/{id}/skipped`) skips one. Both work only
   before the occurrence starts; once it is running you cannot un-run it.
5. **Amend rather than delete.** `updateMaintenanceWindow` (`PATCH /v1/maintenance/windows/{id}`)
   changes the schedule in place. `deleteMaintenanceWindow` is permanent with no restore.

## Consequences

Cancel and skip act on a single occurrence and leave the schedule intact — prefer them over deleting
the window. Deleting the window removes every future occurrence and cannot be undone.

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
