---
name: moogsoft-manage-api-keys
description: Create, inspect and revoke Moogsoft API keys with least privilege, including the permission rules that govern what a key can ever do.
api: Moogsoft User Management API
operations:
  - createApiKey
  - getUserApiKeys
  - getApiKey
  - getAllApiKeys
  - deleteApiKey
  - deleteMultipleApiKeys
---

# Manage Moogsoft API keys

## Steps

1. **Understand the ceiling first.** A key can never carry more permission than the user who created
   it, and it always belongs to its creator even when another person uses it. Decide which user
   should own the key before you create it.
2. **Create.** `createApiKey` (`POST /v2/users/{userId}/keys`). Grant only the feature areas the
   integration needs, and choose Read Only unless a write is genuinely required.
3. **Capture the secret immediately.** The key value is returned once, at creation. `getApiKey`
   (`GET /v2/users/{userId}/keys/{keyId}`) returns metadata only — the secret is never readable
   again.
4. **Inventory.** `getUserApiKeys` (`GET /v2/users/{userId}/keys`) per user,
   `getAllApiKeys` (`GET /v1/keys`) across the instance.
5. **Revoke.** `deleteApiKey` (`DELETE /v2/users/{userId}/keys/{keyId}`) or
   `deleteMultipleApiKeys` (`DELETE /v1/keys`). Revocation propagates in roughly five minutes, after
   which the key is permanently dead — there is no re-issue and no undo.

## Notes worth carrying

- One key per data stream is the vendor's own recommendation for inbound integrations.
- Creating an integration needs Full Access to Integrations for the creating call, but the key the
  integration then USES should be Read Only, or hold no permissions at all.
- The ServiceNow outbound integration needs broader permissions than other integrations.
- Keys created by a deleted user survive that user's deletion — audit `getAllApiKeys` after offboarding.

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
