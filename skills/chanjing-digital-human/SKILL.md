---
name: chanjing-digital-human
description: Generate Chanjing digital-human videos via website-project synthesis and wire them into FrameVideo compositions. Use when listing public/custom digital humans or voices, checking Chanjing OAuth login, submitting/polling synthesis tasks, or wiring assets into compositions. For local Kokoro TTS fallback see framevideo-media; for composition HTML conventions see framevideo.
---

# Chanjing Digital Human

Access Chanjing website-side digital humans, voices, and synthesis tasks from FrameVideo. Reuse the OAuth plugin API client in `packages/cli/src/tts/chanjingOpenapi.ts` and the shared auth store in `packages/cli/src/auth/store.ts` — do not copy standalone credential scripts.

## Auth Contract

1. `framevideo auth login` or Studio login starts OAuth CLI Web Login.
2. Tokens live in the shared `oauth` block in `~/.chanjing/credentials` (honors `CHANJING_CONFIG_DIR`).
3. Plugin requests use `Authorization: Bearer <access_token>`.
4. Do not use `app_id`, `secret_key`, `CHANJING_OPENAPI_ACCESS_TOKEN`, or `dp_open_app` data.

```bash
framevideo auth status
```

Studio auth routes: see [studio-routes.md](./references/studio-routes.md).

## Missing Credentials UX

When OAuth is missing during an agent-authored workflow, do not stop at env-var guidance if preview is available:

1. Start or reuse `npx framevideo preview`.
2. Open the Studio project URL in the in-app browser.
3. Navigate to Digital Human, Voice, or account/login panel.
4. Click login to start OAuth CLI Web Login.
5. Let the user complete browser authorization.
6. Re-check `GET /api/projects/:id/chanjing/auth/status`, then continue.

Use CLI-only guidance only when the browser UI is unavailable or the user requests non-interactive setup. Never print or save tokens outside the shared credential store.

## Hard Rules

### `person_id` and `voice_id` sources

| Who picks | `person_id` | `voice_id` |
| --------- | ----------- | ---------- |
| **User provides** explicit ids | Use the provided id, but fetch that person's real `figure_type`, `width`, and `height` from `GET /api/projects/:id/digital-humans/common` or `POST /api/projects/:id/digital-humans/custom` before layout | Use directly |
| **Agent selects** | Must come from `GET /api/projects/:id/digital-humans/common` or `POST /api/projects/:id/digital-humans/custom` after OAuth | Must come from `GET /api/projects/:id/tts/voices` (optionally filtered by tags), or the selected public person's `audioManId` |

When the agent selects:

- Do not invent ids, use fixtures, copied payloads, plugin-client list results, or text-generation status ids.
- Text-generated people: wait until the person appears in `/digital-humans/custom` before using that id.
- Compare `name`, `cover`, `previewVideoUrl`, `audioManId`, `figureType`, `width`, `height`, and `digital_person_type`; do not pick the first candidate blindly.
- Tag filtering for public resources: see [studio-routes.md](./references/studio-routes.md).

When the user provides ids, keep the provided `person_id` and `voice_id` as authoritative. Still fetch the matching digital-human record before layout so `figure_type`, `width`, and `height` reflect the real person; if no matching person exists, report that error instead of falling back to `whole_body`.

### Other non-negotiables

- No direct OpenAPI digital-human generation — `createDigitalHumanVideo()` returns 501.
- Never pass a digital-human id as `projectId`.
- Never hand-author `workspace_v2`.
- Download generated video to `assets/digital-humans/` before referencing in composition HTML.
- Do not use render-time network URLs in compositions.

## Standard Workflow

1. **Auth** — check status; if missing, run Missing Credentials UX above.
2. **Resolve ids** — user-provided ids → keep ids authoritative but fetch the matching person metadata for `figure_type`, `width`, and `height`; agent selection → list via Studio routes (people + voices).
3. **Layout** — derive direction/canvas from real person dimensions; full-canvas presenter by default. See [website-project-payload.md](./references/website-project-payload.md).
4. **Save** — `saveWebsiteProject(payload)`; verify `project_id` is returned.
5. **Submit** — `submitWebsiteVideo({ projectId: saved.project_id })`; verify `task_id`.
6. **Poll** — `getDigitalHumanVideo(taskId)` until finished or terminal failure. See [api-enums.md](./references/api-enums.md).
7. **Download + wire** — save to `assets/digital-humans/`, then wire into composition (below).

## End-to-End TypeScript Pattern

```ts
import { mkdir, writeFile } from "node:fs/promises";
import { join } from "node:path";
import { ChanjingOpenApiClient } from "../packages/cli/src/tts/chanjingOpenapi.js";

const client = new ChanjingOpenApiClient();

// Agent selection example — skip listing when user already supplied person_id / voice_id
const commonRes = await fetch(`/api/projects/${projectId}/digital-humans/common`);
const { digitalHumans } = await commonRes.json();
const person = digitalHumans.find((p) => p.audioManId);
if (!person) throw new Error("No suitable digital human found");

const voicesRes = await fetch(`/api/projects/${projectId}/tts/voices`);
const { voices } = await voicesRes.json();
const voiceId = person.audioManId ?? voices[0]?.id;
if (!voiceId) throw new Error("No voice available");

const isVertical = (person.height ?? 0) > (person.width ?? 0);
const canvas = isVertical
  ? { width: 1080, height: 1920 }
  : { width: 1920, height: 1080 };

const payload = {
  project_id: "",
  name: "产品介绍口播",
  direction: isVertical ? "vertical" : "horizontal",
  canvas,
  scenes: [
    {
      key: "scene-1",
      duration: 8,
      background: { type: "color", color: "#ffffff" },
      speech: { text: "大家好，欢迎了解我们的产品。", voice_id: voiceId, show_subtitles: false },
      tracks: [
        {
          key: "presenter",
          type: "person",
          z_index: 10,
          elements: [
            {
              key: "person-1",
              person_id: person.id,
              source: "common",
              figure_type: person.figureType ?? "whole_body",
              x: 0,
              y: 0,
              width: canvas.width,
              height: canvas.height,
              mouth_mode: 256,
              backway: 2,
            },
          ],
        },
      ],
    },
  ],
};

const saved = await client.saveWebsiteProject(payload);
if (!saved.project_id) throw new Error("Chanjing did not return project_id");

const submitted = await client.submitWebsiteVideo({ projectId: saved.project_id });
if (!submitted.task_id) throw new Error("Chanjing did not return task_id");

const TERMINAL_FAIL = new Set([40, 41, 50, 1101, 999]);
let videoUrl: string | undefined;
for (let i = 0; i < 180; i++) {
  const video = await client.getDigitalHumanVideo(submitted.task_id);
  if (video.status === 30 && video.video_url) {
    videoUrl = video.video_url;
    break;
  }
  if (video.status !== undefined && TERMINAL_FAIL.has(video.status)) {
    throw new Error(video.msg ?? `Task failed: status ${video.status}`);
  }
  await new Promise((r) => setTimeout(r, 2000));
}
if (!videoUrl) throw new Error(`Timed out waiting for task ${submitted.task_id}`);

const outDir = join(projectDir, "assets", "digital-humans");
await mkdir(outDir, { recursive: true });
const assetPath = join(outDir, "presenter-intro.mp4");
const res = await fetch(videoUrl);
await writeFile(assetPath, Buffer.from(await res.arrayBuffer()));
```

Reuse existing repo helpers for slugging, polling, or downloading when editing `studioServer.ts`.

## Composition Wiring

Keep the visual `<video>` muted; add a separate `<audio>` clip for voiceover timing. See the `framevideo` skill for clip conventions.

```html
<video class="clip" src="assets/digital-humans/presenter-intro.mp4"
  data-start="0" data-duration="8" data-track-index="1" muted playsinline></video>
<audio class="clip" src="assets/digital-humans/presenter-intro.mp4"
  data-start="0" data-duration="8" data-track-index="10" data-volume="1"></audio>
```

- Never add silence placeholders for generated digital-human audio.
- Transparent presenter: invoke `framevideo-media` remove-background after download.
- Captions: use synthesis timing fields when available; otherwise invoke `framevideo-media` transcribe.

## Safety

- Never print access tokens or credential-store contents.
- Do not commit generated credentials or local API keys.
- Avoid duplicating standalone `chanjing-video-compose` Python auth scripts.

## References

- [website-project-payload.md](./references/website-project-payload.md) — field reference, layout defaults, minimal JSON
- [studio-routes.md](./references/studio-routes.md) — Studio routes, plugin client, tag filtering
- [api-enums.md](./references/api-enums.md) — status codes and polling logic
