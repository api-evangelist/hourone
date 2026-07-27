---
name: Preview a voice, then build a dynamic video
description: >-
  Audition a text-to-speech voice with a short preview, then build a video
  dynamically from scenes you define, and add subtitles once it is ready.
api: openapi/hourone-openapi-original.json
base_url: https://api.makereals.com/api/v1
auth: api-key header
operations:
  - create_audio_preview_voice_preview_post
  - get_voice_preview_voice_preview__task_id__get
  - create_video_videos_post
  - get_video_by_id_videos__video_id__get
  - create_subtitles_videos__video_id__subtitles_post
---

# Preview a voice, then build a dynamic video

Grounded in real operationIds from `openapi/hourone-openapi-original.json`. Base URL
`https://api.makereals.com/api/v1`; `api-key` header on every call.

## Steps

1. **Request a voice preview** — `create_audio_preview_voice_preview_post` →
   `POST /voice-preview`. Send `source_draft_id` (a draft containing the chosen
   voice) and `transcript` (up to 500 characters). Returns a task `id` and `status`.

2. **Poll the preview** — `get_voice_preview_voice_preview__task_id__get` →
   `GET /voice-preview/{task_id}`. `status` (`StatusTypes`) is
   `in_progress → ready` (or `failed`); when `ready`, read the audio `url`.

3. **Create the video dynamically** — `create_video_videos_post` → `POST /videos`.
   Build `scenes[]` from scratch: each scene carries `media`, `texts`, and either a
   `transcript` (text-to-speech) or a public `voice_recording_url` (a `.wav` URL).
   Set `character_id` / `voice_id` at the root or per scene (per-scene overrides the
   root). Match `palette` to the template size.

4. **Wait for the video** — `get_video_by_id_videos__video_id__get` →
   `GET /videos/{video_id}` until `status == ready`.

5. **Add subtitles (optional)** — `create_subtitles_videos__video_id__subtitles_post`
   → `POST /videos/{video_id}/subtitles`. Only works **after** the video is `ready`.
   Choose format `vtt` or `srt` and a supported language code
   (en, es, de, pt, it, fr, nl, ja, tr, pl, hu, cz, he, zh, hi, ru, ko, ar).
   Response returns the subtitles file `url`.

## Rules

- `transcript` for a voice preview is capped at **500 characters**.
- `voice_recording_url` must be a **publicly reachable `.wav`**; when provided, the
  scene `transcript` is optional.
- Subtitles before the video is `ready` fail — gate on status first.
- All validation errors are HTTP `422` (`errors/hourone-problem-types.yml`).
