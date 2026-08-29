---
name: moogsoft-configure-outbound-webhooks
description: Stand up a Moogsoft outbound webhook that pushes incident and alert changes to an external system, rehearsing it before it goes live and reading its delivery logs afterwards.
api: Moogsoft Webhook API
operations:
  - createWebhookV2
  - previewWebhookV2
  - testWebhookV2
  - getWebhooksV2
  - getWebhookV2
  - patchWebhookV2
  - getWebhookLogs
  - getWebhookMetrics
  - deleteWebhookV2
---

# Configure a Moogsoft outbound webhook

## Steps

1. **Render before you send.** `previewWebhookV2` (`POST /v2/integrations/webhooks/preview`) shows
   the exact payload Moogsoft would deliver for a given configuration. Nothing is saved and nothing
   is sent.
2. **Fire a rehearsal.** `testWebhookV2` (`POST /v2/integrations/webhooks/test`) performs a real test
   delivery against your endpoint without persisting the webhook.
3. **Create it.** `createWebhookV2` (`POST /v2/integrations/webhooks/items`). Outbound auth to your
   endpoint supports OAuth 2.0 (password grant since November 2024, client credentials via the
   credential store since April 2025), basic, and header tokens.
4. **Verify delivery.** `getWebhookLogs` (`GET /v2/integrations/webhooks/logs/{id}`) is the
   per-webhook delivery log; `getWebhookMetrics` (`GET /v2/integrations/webhooks/metrics/{id}`) is
   the counter set. Check both after the first real incident rather than assuming success.
5. **Amend in place.** `patchWebhookV2` (`PATCH /v2/integrations/webhooks/items/{id}`) for partial
   changes, `modifyWebhookV2` (`PUT`) for a full replacement.

## Consequences and gaps

`deleteWebhookV2` is permanent and takes the configuration with it — `getWebhookV2` first and keep
the payload if you may need to recreate it. Moogsoft publishes **no signing scheme** for outbound
deliveries and **no documented retry policy**, so the receiving endpoint must authenticate the
caller by other means and must tolerate duplicates.

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
