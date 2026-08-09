---
name: Preview a voice then generate a dynamic video with subtitles
description: Preview synthesized narration audio, then create a dynamic scene-based video and attach translated subtitles.
api: openapi/hour-one-openapi.json
operations: [create_audio_preview_voice_preview_post, get_voice_preview_voice_preview__task_id__get, create_video_videos_post, create_subtitles_videos__video_id__subtitles_post]
generated: '2026-07-19'
method: generated
---

# Preview a voice, then generate a dynamic video with subtitles

Validate narration before spending render time, generate a dynamic (scene-defined) video, then add subtitles.

## Auth
- Base URL: `https://api.makereals.com/api/v1`
- Send the `api-key: <your key>` header on all operations below.

## Steps
1. Start a voice preview with `create_audio_preview_voice_preview_post` (`POST /voice-preview`), supplying the narration text and `voice_id`. It returns a `task_id`.
2. Poll `get_voice_preview_voice_preview__task_id__get` (`GET /voice-preview/{task_id}`) until the task status is `ready` (states: `in_progress`, `ready`, `failed`); the response yields the preview audio.
3. Create the dynamic video with `create_video_videos_post` (`POST /videos`), defining `scenes` (each with `transcript`, `media_elements`, `texts_elements`, `layout_id`) and the chosen `voice_id` / `character_id`. Capture the returned video `id`.
4. Once the video status is `ready`, add subtitles with `create_subtitles_videos__video_id__subtitles_post` (`POST /videos/{video_id}/subtitles`), choosing a `language_code` (en, es, de, pt, it, fr, nl, ja, tr, pl, hu, cz, he, zh, hi, ru, ko, ar) and `format`.

## Rules
- Voice preview is the cheap check — do it before generating full videos in a loop.
- Subtitles require a finished video; sequence step 4 after the video reaches `ready`.
- Validation errors return HTTP 422 with the FastAPI `detail[]` envelope.
