---
name: chanjing-digital-human
description: Chanjing platform integration for FrameVideo: OAuth-aware digital-human video synthesis, public/custom digital-human and voice selection, Chanjing AI Creation image/video assets, platform BGM/SFX downloads, polling/downloading generated assets, and wiring Chanjing assets into FrameVideo compositions. Use when listing Chanjing people or voices, checking Chanjing auth in a platform workflow, submitting/polling digital-human videos, planning/submitting/downloading AI Creation image/video tasks, downloading Chanjing music or sound effects, or integrating Chanjing-generated assets into FrameVideo. For local Kokoro TTS/transcribe/remove-background use framevideo-media; for composition HTML conventions use framevideo.
---

# Chanjing Platform for FrameVideo

## When To Use

Use this skill for:

- **数字人视频** — 生成蝉镜平台数字人主持视频
- **AI Creation 资产** — 使用蝉镜 AIGC 生成图片/视频素材
- **平台音乐/音效** — 下载蝉镜平台背景音乐和音效
- **OAuth 认证** — 检查/设置蝉镜平台认证
- **资产集成** — 将蝉镜生成的资产接入 FrameVideo composition

## Do NOT Use

Avoid this skill for:

- **本地 TTS** — 使用 `framevideo-media` (Kokoro TTS)
- **本地音频处理** — 使用 `framevideo-media` (transcribe, remove-background)
- **Composition HTML 编写** — 使用 `framevideo`
- **SSML 语音标记** — 使用 `framevideo-voiceover-ssml`

---

## Quick Start

### 场景 1: 生成数字人视频

```bash
# 1. 检查认证
npx framevideo auth status

# 2. 列出可用数字人和声音
npx framevideo chanjing people list --json
npx framevideo chanjing voices list --json

# 3. 创建数字人视频项目
# (详见 references/website-project-payload.md)

# 4. 下载生成的视频
# 下载到 assets/digital-humans/
```

### 场景 2: 下载平台音乐

```bash
# 1. 认证
npx framevideo auth login

# 2. 浏览音乐分类
npx framevideo chanjing music categories --json

# 3. 列出音乐
npx framevideo chanjing music list --category <id> --compact

# 4. 下载
npx framevideo chanjing music download --id <id> --output assets/music/bg.mp3
```

### 场景 3: AI Creation 生图/生视频

```bash
# 使用 AI Creation API 生成图片或视频
# (详见 references/ai-creation.md)
```

---

## Routing

Read only what the request needs:

| User intent | Read |
| --- | --- |
| Auth status, login, missing credentials, Studio login routes | [studio-routes.md](./references/studio-routes.md) |
| Select/list public or custom digital humans or voices | [studio-routes.md](./references/studio-routes.md) |
| Generate digital-human presenter video | [website-project-payload.md](./references/website-project-payload.md), then [api-enums.md](./references/api-enums.md) |
| Poll digital-human task status or interpret status codes | [api-enums.md](./references/api-enums.md) |
| Generate Chanjing AI image/video assets | [ai-creation.md](./references/ai-creation.md) |
| Wire generated presenter, AI B-roll, images, captions, or local assets into FrameVideo | [composition-wiring.md](./references/composition-wiring.md) |
| Download Chanjing BGM or SFX | [studio-routes.md](./references/studio-routes.md), section “Background music and sound effects” |
| Use Shot plans from `framevideo-ai-production` for real AIGC generation | [ai-creation.md](./references/ai-creation.md) plus `framevideo-ai-production/references/chanjing-aigc-handoff.md` |

For local-only Kokoro TTS, Whisper transcription, or background removal, use `framevideo-media` instead.

## Platform Rules

- Use OAuth Bearer tokens from the shared Chanjing credential store. Never print, echo, save, or commit tokens.
- Prefer the existing FrameVideo CLI, Studio routes, and `ChanjingOpenApiClient` in `packages/cli/src/tts/chanjingOpenapi.ts`; do not copy standalone credential scripts.
- Do not use `app_id`, `secret_key`, `CHANJING_OPENAPI_ACCESS_TOKEN`, or `dp_open_app` data for agent-authored workflows.
- If OAuth is missing and preview/Studio is available, prefer the Studio login UX instead of stopping at env-var instructions.
- Download every generated or selected platform asset before referencing it in composition HTML. Do not use render-time remote Chanjing/OSS URLs.

## Asset Locations

Use these local project paths:

| Asset | Local directory |
| --- | --- |
| Digital-human videos | `assets/digital-humans/` |
| AI Creation images | `assets/ai-creation/images/` |
| AI Creation videos | `assets/ai-creation/videos/` |
| Chanjing background music | `assets/music/` |
| Chanjing sound effects | `assets/sfx/` |

## Digital-Human Workflow

1. Check auth. If missing, use Studio login flow from [studio-routes.md](./references/studio-routes.md).
2. Resolve `person_id` and `voice_id`.
3. Fetch the selected person's real metadata before layout.
4. Build a website-project payload from [website-project-payload.md](./references/website-project-payload.md).
5. Save the website project and verify `project_id`.
6. Submit video synthesis and verify `task_id`.
7. Poll using [api-enums.md](./references/api-enums.md).
8. Download the finished video to `assets/digital-humans/`.
9. Wire it into FrameVideo using [composition-wiring.md](./references/composition-wiring.md).

### ID Sources

| Who picks | `person_id` | `voice_id` |
| --- | --- | --- |
| User provides explicit ids | Use the provided id, but fetch matching person metadata before layout | Use directly |
| Agent selects | Must come from current Studio/CLI Chanjing list routes after OAuth | Must come from current Chanjing voice list routes, or selected public person's `audioManId` |

Do not invent ids or reuse fixture ids. If a user-provided person id cannot be fetched, report that error instead of falling back to another person.

## AI Creation Workflow

Use this for non-presenter AIGC images/videos such as B-roll, generated backgrounds, concept images, product display clips, transition clips, or abstract visuals.

1. Read [ai-creation.md](./references/ai-creation.md).
2. Fetch active models with `framevideo chanjing ai-models ...`; do not read backend constants or copied model lists.
3. Build payloads from the selected model's `params_config.fields`; do not hard-code model options.
4. Create `.framevideo/ai-creation-tasks/*.json` task metadata before submission.
5. Apply local and remote idempotency checks before submitting billable tasks.
6. Submit only missing tasks.
7. Use short sync runs to poll/download.
8. Compose only with local files under `assets/ai-creation/images/` or `assets/ai-creation/videos/`.

## Music and SFX Workflow

Use Chanjing platform music/SFX when the user asks for platform BGM, soundtrack, chorus extraction, whooshes, hits, clicks, or transition sounds.

```bash
npx framevideo chanjing music categories --json
npx framevideo chanjing music list --compact
npx framevideo chanjing music download --id <music-id> --chorus --duration 10 --json

npx framevideo chanjing sound-effect list --compact
npx framevideo chanjing sfx download --id <effect-id> --volume 0.8 --json
```

Keep BGM under `assets/music/` and SFX under `assets/sfx/`. Use low BGM volume under speech or digital-human narration.

## Composition Wiring

Read [composition-wiring.md](./references/composition-wiring.md) before writing HTML that uses Chanjing assets.

Core rules:

- Visual video clips are muted.
- Add a separate `<audio>` clip when the generated video contains narration.
- Keep captions clear of faces, hands, presenter body, CTA, product UI, and platform subtitles.
- For styled FrameVideo captions, prefer synthesis timing fields when available; otherwise use `framevideo-media` to transcribe the downloaded local video/audio.
- For transparent presenter overlays, use `framevideo-media` remove-background after download.

## Hard Stops

- Do not pass a digital-human id as a website `projectId`.
- Do not hand-author `workspace_v2`.
- Do not call compatibility stubs such as direct digital-human generation routes that return 501.
- Do not insert remote `output_url` or Chanjing/OSS URLs into composition HTML.
- Do not create duplicate AI Creation tasks when matching local or recent remote task metadata exists.

## Validation

After wiring Chanjing assets into a FrameVideo project:

```bash
npx framevideo lint
npx framevideo validate
npx framevideo inspect
```

For visual overlap, caption safety, presenter placement, and B-roll readability, use `framevideo-visual-qa`.
