# Website Project Payload

There is no direct "create digital-human video from person id + text" API. Use the website-project flow so the draft can be opened on the Chanjing website:

1. `saveWebsiteProject(payload)` — `/plugin/v1/chanjing/projects/save`
2. `submitWebsiteVideo({ projectId, ... })` — `/plugin/v1/chanjing/videos/submit`
3. `getDigitalHumanVideo(taskId)` — `/plugin/v1/chanjing/videos/status?task_id=...`

Do not use `createDigitalHumanVideo(options)` (501 stub). Do not hand-author `workspace_v2`; the backend converts the simplified DTO.

## Default layout

- Derive `direction` and canvas from the selected person's `width`/`height`: if `height > width` → `vertical` + `1080×1920`; if `width >= height` → `horizontal` + `1920×1080`. If dimensions are missing, default horizontal.
- Unless the user asks for side layout, PIP, avatar crop, or a specific box, make the presenter element fill the whole canvas (`x:0, y:0, width:canvas.width, height:canvas.height`).
- Generate a clean full-canvas source video first; crop/scale/mask in FrameVideo composition HTML if the final edit needs split layout.
- Talking-head drafts: set presenter `backway: 2` unless the user wants forward-only playback.

## Top-level fields

| Field | Required | Notes |
| ----- | -------- | ----- |
| `project_id` | no | Omit or `""` to create; pass returned id to edit |
| `name` | yes | Project title, max 200 runes |
| `direction` | no | `horizontal` \| `vertical`; set explicitly for talking-head |
| `canvas` | no | `{ width, height }`; defaults per direction |
| `scenes` | yes | Ordered scene list |

## Scene fields

| Field | Required | Notes |
| ----- | -------- | ----- |
| `key` | no | Stable scene key; backend generates if omitted |
| `duration` | no | Seconds; default 5 |
| `background` | no | `type=color` + `color`, or `type=image/video` + `resource_id`/`url` |
| `speech` | yes | `text` and `voice_id` required |
| `tracks` | no | Ordered visual tracks |

## Speech fields

| Field | Default | Notes |
| ----- | ------- | ----- |
| `text` | — | Required TTS/口播 text |
| `plain_text` | `text` | Plain fallback |
| `voice_id` | — | Required website voice id |
| `speed` | `1` | |
| `pitch` | `1` | |
| `volume` | `100` | |
| `show_subtitles` | `false` | Prefer FrameVideo captions; see Subtitles below |

## Track and element fields

Set `track.type` explicitly when possible (`person`, `image`/`pic`, `video`, `text`, `background`). Backend stores image tracks as website type `pic`.

Presenter element minimum: `person_id`, `source`, `x`, `y`, `width`, `height`.

| Field | Notes |
| ----- | ----- |
| `person_id` | Website digital-human id |
| `source` | `common`, `custom`, `text`, `user` — hint only |
| `figure_type` | From list response `figureType`; common people may require it; custom default `origin` |
| `mouth_mode` | `256` \| `512` \| `768`; default `256` |
| `backway` | `1` forward; `2` 正反播 — default `2` for talking-head |
| `alpha` | Default `1` |
| `resource_id` / `url` | For image/video/background elements |
| `text`, `color`, `font_size`, `x`, `y`, `width`, `height` | For text elements |

## `submitWebsiteVideo` fields

| Field | Required | Notes |
| ----- | -------- | ----- |
| `projectId` | yes | From `saveWebsiteProject` — never a digital-human id |
| `type` | no | Default `make` |
| `version` | no | |
| `model` | no | `1.0` \| `2.0`; omit unless required |
| `templateId` | no | |

## Draft requirements

- Always include at least one scene with `speech.text` and `speech.voice_id`.
- One semantic object type per track.
- Do not hand-author `workspace_v2.subtitle_config`; set `show_subtitles: false` or omit for ordinary drafts.
- Canvas-pixel coordinates; positive width/height for visible elements.
- Save first, verify `project_id`, then submit.

## Minimal full-canvas payload

```json
{
  "project_id": "",
  "name": "产品介绍口播",
  "direction": "horizontal",
  "canvas": { "width": 1920, "height": 1080 },
  "scenes": [
    {
      "key": "scene-1",
      "duration": 8,
      "background": { "type": "color", "color": "#ffffff" },
      "speech": {
        "text": "大家好，欢迎了解我们的产品。",
        "voice_id": "voice-id-from-user-or-selection",
        "show_subtitles": false
      },
      "tracks": [
        {
          "key": "presenter",
          "type": "person",
          "z_index": 10,
          "elements": [
            {
              "key": "person-1",
              "person_id": "digital-human-id",
              "source": "common",
              "figure_type": "whole_body",
              "x": 0,
              "y": 0,
              "width": 1920,
              "height": 1080,
              "mouth_mode": 256,
              "backway": 2,
              "alpha": 1
            }
          ]
        }
      ]
    }
  ]
}
```

## Subtitles

- Ask show/hide before task creation unless the workflow already decided.
- Default hidden for non-interactive workflows.
- For styled FrameVideo captions: hide Chanjing platform subtitles, then use generation timing (`subtitle_data_url`, `video_text`, word/sentence transcript) or invoke `framevideo-media` transcribe on the downloaded audio.

## Response field mapping

Studio list responses use camelCase; payload uses snake_case:

| List response | Payload field |
| ------------- | ------------- |
| `figureType` | `figure_type` |
| `audioManId` | `speech.voice_id` (when using the person's bundled voice) |
