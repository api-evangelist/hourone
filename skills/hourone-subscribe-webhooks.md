---
name: Subscribe to video webhooks and verify signatures
description: >-
  Register a webhook to receive video.ready / video.failed notifications instead of
  polling, and verify each delivery's HMAC-SHA256 signature.
api: openapi/hourone-openapi-original.json
base_url: https://api.makereals.com/api/v1
auth: api-key header
operations:
  - create_webhook_webhooks_post
  - get_webhooks_webhooks_get
  - regenerate_webhook_secret_webhooks__webhook_id__secret_put
---

# Subscribe to video webhooks

Grounded in real operationIds from `openapi/hourone-openapi-original.json`. Base URL
`https://api.makereals.com/api/v1`; `api-key` header on every call. Full event
catalog in `asyncapi/hourone-webhooks.yml`.

## Steps

1. **Create a webhook** — `create_webhook_webhooks_post` → `POST /webhooks`.
   Send `name`, `url` (an HTTPS endpoint you control), and `events` — choose from
   `video.ready` and `video.failed`. The response returns the webhook `id` and a
   `signing_secret`; store the secret securely (it is used to verify deliveries).

2. **List / confirm** — `get_webhooks_webhooks_get` → `GET /webhooks` to confirm the
   endpoint is registered and its `status` is `active`.

3. **Verify every delivery.** Each POST to your URL includes header
   `x-hourone-signature`. Compute `HMAC-SHA256(secret, raw_body)` as hex and compare
   (constant-time) to that header; reject on mismatch. The body is
   `{ "event_name": "...", "data": { "id", "req_id", "draft_id", "status" } }`.

4. **Rotate the secret if leaked** —
   `regenerate_webhook_secret_webhooks__webhook_id__secret_put` →
   `PUT /webhooks/{webhook_id}/secret`.

## Rules

- Always use an **HTTPS** callback URL.
- **Verify the signature on every request** before trusting the payload.
- Endpoints that fail repeatedly move to `inactive`; re-enable via the pause/toggle
  operations. Webhooks replace tight polling of `GET /videos/{video_id}`.
