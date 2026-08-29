---
name: moogsoft-triage-incident
description: Work a Moogsoft incident end to end — read it, read its alerts and timeline, check probable root cause and similar past incidents, comment, assign and update status.
api: Moogsoft Alerts/Incidents API, Probable Root Cause API, Similar Incidents API
operations:
  - listIncidents
  - getIncident
  - getSituationRoomAlerts
  - getIncidentTimeline
  - getIncidentPrc
  - getIncidentSimilarity
  - addComment
  - updateIncident
  - updateIncidentTags
---

# Triage a Moogsoft incident

## Steps

1. **Find the work.** `listIncidents` (`GET /v1/incidents`) or `listIncidentsPost`
   (`POST /v1/incidents`) with a `json_filter`. Use `start` + `limit`, and `search_after` for deep
   paging. Never assume you got everything — the cap is 1000 records per call.
2. **Read the incident.** `getIncident` (`GET /v1/incidents/{incidentId}`).
3. **Read its alerts.** `getSituationRoomAlerts` (`POST /v1/situation-room/{incidentId}/alerts`)
   returns the correlated alerts. `getIncidentTimeline`
   (`GET /v1/situation-room/{incidentId}/timeline`) returns how it unfolded.
4. **Ask the platform what it thinks.** `getIncidentPrc`
   (`GET /v1/root-cause/incidents/probabilities/{id}`) returns Moogsoft's probable root cause and
   alert labels. `getIncidentSimilarity` (`POST /v1/incident-similarity`) returns similar past
   incidents — the fastest route to a known fix.
5. **Record what you found.** `addComment` (`POST /v1/incidents/{incidentId}/comments`). Comments
   are the audit trail; write the reasoning, not just the conclusion.
6. **Act.** `updateIncident` (`PATCH /v1/incidents/{incidentId}`) sets `status`, `assignee`,
   `assigned_groups`, `description`, `priority`, `impact`, `urgency`. `updateIncidents`
   (`PATCH /v1/incidents`) does the same across a list of `ids`. `updateIncidentTags`
   (`PATCH /v1/incidents/{incidentId}/tags`) OVERWRITES tags — read them first if you mean to add.
7. **Link the ticket.** `postIncidentExternalId`
   (`POST /v1/incidents/{incidentId}/external-ids/{integrationId}`) records the ServiceNow / Jira
   identity on the incident.

## Consequences

Status and assignment changes are visible to responders immediately and are not versioned.
`deleteComment` is permanent. Tag updates replace rather than merge.

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
