---
name: Create a video from a Blueprint and wait for it
description: >-
  Generate an AI video from an existing HourOne studio project (Blueprint), then
  poll until it is ready and retrieve the download URL. Uses the async
  create-then-poll pattern.
api: openapi/hourone-openapi-original.json
base_url: https://api.makereals.com/api/v1
auth: api-key header
operations:
  - create_video_videos_post
  - get_video_by_id_videos__video_id__get
---

# Create a video from a Blueprint

Grounded in real operationIds from `openapi/hourone-openapi-original.json`. Base URL
`https://api.makereals.com/api/v1`. Every request sends the `api-key` header.

## Steps

1. **Authenticate.** Set header `api-key: <YOUR_KEY>`. Only one key is active at a
   time (see `authentication/hourone-authentication.yml`). Never send the key in a
   query string.

2. **Create the video** — `create_video_videos_post` → `POST /videos`.
   Send a Blueprint request whose `type` targets an existing studio project:
   provide `source_draft_id` (or `template_id`) plus any overrides
   (`palette_id`, `character_id`, `voice_id`, `folder_id`) and per-scene `scenes[]`
   with `texts`/`transcript`. Optionally set `correlation_id` to group the video for
   analytics. The DraftID is found in the project editor URL at app.hourone.ai.
   Response `201` returns a `VideoResponse` with `id` and `status`.

3. **Poll for completion** — `get_video_by_id_videos__video_id__get` →
   `GET /videos/{video_id}`. Poll using the `id` from step 2. `status`
   (`VideoStatusEnum`) moves `not_started → started → processing → ready`
   (or `failed`). Use `progress` (0–100) for UX. Prefer webhooks over tight polling
   when possible (see `skills/hourone-subscribe-webhooks.md`).

4. **Retrieve output.** When `status == ready`, read `download_url` and
   `video_player_url` from the response.

## Rules

- **Async, not synchronous** — creation returns immediately with a task; the video
  is not ready at `201`. Poll or use a webhook.
- **Palette size must match the template**, or `POST /videos` returns `422` with the
  expected palette size (`errors/hourone-problem-types.yml`).
- **No idempotency key** is supported; do not blindly retry `POST /videos` on
  timeout — first `GET /videos` to check whether the prior call already created one.
- Validation failures return HTTP `422` with `{ "detail": [ ... ] }`.
