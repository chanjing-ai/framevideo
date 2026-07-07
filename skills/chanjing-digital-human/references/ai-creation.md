# AI Creation and B-roll

Use this reference when generating Chanjing AI image/video assets or planning B-roll for a FrameVideo composition.

## Model and task rules

- Image generation submits `creation_type: 3`; video generation submits `creation_type: 4`.
- Fetch available models from the active FrameVideo CLI or active preview server. Do not read backend constants, fixtures, copied payloads, or handwritten model lists.
- The CLI result is the source of truth because it uses the active FrameVideo build config and the current OAuth account.
- Build payloads from the selected model's `params_config.fields`; do not hard-code duration, clarity, aspect ratio, reference requirements, or model-specific options.
- Do not rely on user payload attribution fields. AI Creation task submission must be marked with `platform: "framevideo"` by the FrameVideo client; ignore user-provided `source`, `client_id`, and `clientId`.

```bash
npx framevideo chanjing ai-models all --project ./my-video --compact
npx framevideo chanjing ai-models image --project ./my-video --compact
npx framevideo chanjing ai-models video --project ./my-video --compact
```

## Agent-friendly commands

```bash
npx framevideo chanjing ai-create image --project ./my-video --prompt "科技发布会背景" --payload ./payload.json
npx framevideo chanjing ai-create video --project ./my-video --prompt "产品展示视频" --payload ./payload.json
npx framevideo chanjing ai-task --project ./my-video --id <task-id> --download
```

## Idempotency and retry safety

AI Creation can create billable remote tasks. Make every image/video generation request idempotent.

1. Normalize the request before submission. Build a stable fingerprint from `creation_type`, selected model id/code/name, prompt or `ref_prompt`, aspect ratio, clarity, duration, number of images, seed when present, reference image/resource ids or URLs, and every user-controlled payload field derived from `params_config.fields`. Exclude attribution fields such as `platform`, `source`, `client_id`, and `clientId`.
2. Check local metadata before submitting. Look under the project `.framevideo/` directory for AI Creation task records or create one such as `.framevideo/ai-creation-tasks/<clip-id-or-fingerprint>.json`. If a matching pending, generating, or successful task exists, reuse its task id and poll/download it.
3. Check recent remote tasks before submitting when local metadata is missing or stale. Use the plugin `/ai_creation/task/history` route with `request_fingerprint` and a bounded `page_size` (default 20); do not scan full history. Prefer an existing successful task; otherwise reuse the earliest pending/generating match.
4. Create a local lock or pending metadata file before submission when multiple clips or agents may run concurrently. Do not start another submit for the same fingerprint while the lock exists; wait, read the task id, or inspect remote recent tasks.
5. Submit only after the local and remote checks find no reusable task. Persist task metadata immediately after receiving a task id, including fingerprint, prompt, creation type, model id/code/name, normalized payload, task id, status, output URLs, local download paths, and timestamps.
6. If the submit call times out, returns an empty body, exits without JSON, or does not expose a task id, do not retry blindly. First query recent remote tasks and match by fingerprint. If a match exists, persist and reuse it. Only submit again after proving no matching remote task was created.
7. When duplicate matching tasks already exist, choose one and stop creating more: prefer a successful task with downloadable outputs, otherwise the earliest pending/generating task, otherwise report the failed task. A failed matching task does not authorize another submit unless the user explicitly passes `--force`.

## Long-running AIGC workflow

Do not run a single Codex turn as `submit -> wait many minutes -> download -> compose -> render`. Use restartable phases:

1. **Plan**: create one metadata file per desired asset under `.framevideo/ai-creation-tasks/*.json`. Include `fingerprint`, `label`, `creation_type`, `model_id`, `model_code`, `prompt`, `payloadPath` or `payloadTemplatePath`, dependency fields such as `referenceImageTaskId`, `taskId`, `status`, `outputUrls`, and `assets`.
2. **Submit**: submit only records without `taskId` and whose dependencies are ready. Prefer `framevideo chanjing ai-create ... --no-wait` or a preview-server submit route. Persist `taskId` immediately. If submission times out, query recent remote tasks by fingerprint before retrying.
3. **Sync**: poll existing task ids with a short command, for example `framevideo chanjing ai-task --id <task-id> --download`. Limit each sync run to one or a few checks; update `status`, `statusCategory`, `outputUrls`, `assets`, `error`, and `lastCheckedAt`, then exit.
4. **Compose**: only after every required asset has local paths, wire B-roll into HTML. Never reference remote `output_url` directly.
5. **Validate/render**: run FrameVideo validation and render in separate phases.

If a project provides helper scripts such as `scripts/submit-aigc-plan.mjs` and `scripts/sync-aigc-tasks.mjs`, use them instead of hand-running long polling loops. The scripts must be safe to run repeatedly and must not create another remote task when a matching metadata record already has a task id.

## Preview-server project routes

Use these only through the active FrameVideo preview server:

- `POST /api/projects/:id/chanjing/ai-creation/models`
- `GET /api/projects/:id/chanjing/ai-creation/video-types`
- `POST /api/projects/:id/chanjing/ai-creation/cost`
- `POST /api/projects/:id/chanjing/ai-creation/tasks`
- `POST /api/projects/:id/chanjing/ai-creation/tasks/page`
- `POST /api/projects/:id/chanjing/ai-creation/tasks/:taskId/cancel`
- `POST /api/projects/:id/chanjing/ai-creation/tasks/:taskId/retry`
- `POST /api/projects/:id/chanjing/ai-creation/download`

## Polling and download

- Submit with `submitAiCreationTask` only after local metadata, local lock, and `/ai_creation/task/history` find no matching fingerprint.
- Poll with `listAiCreationTasks({ unique_ids: [taskId] })`; if exact lookup is temporarily empty, fall back to `/ai_creation/task/history` with `unique_ids` before reporting that the task is not visible.
- Treat `Queued`, `Ready`, and `Generating` as in-progress.
- Treat `Success` as done.
- Treat `Error`, `Fail`, and `Cancelled` as terminal failures.
- Read generated results from `output_url[]`.
- Download every successful output before using it:
  - images: `assets/ai-creation/images/`
  - videos: `assets/ai-creation/videos/`
- If submit, polling, or download fails, do not insert remote media into composition. Report the task id and failure reason.

## B-roll workflow

Use this when narration, subtitles, digital-human presenter video, or an existing composition needs supporting visuals.

1. Analyze the A-roll: read the script, captions, digital-human speech, or composition and split it into semantic segments with `start`, `end`, `speech`, and `visual_intent`. If captions exist, align B-roll to caption timings; if not, estimate timings and generate or transcribe captions when needed.
2. Pick B-roll only where visuals clarify the message or improve pacing: product capabilities, workflow steps, abstract concepts, data changes, user pain points, transitions, examples, and final payoff. Do not add B-roll to every sentence.
3. Create a plan for each clip: `id`, `start`, `duration`, `type` (`image`, `video`, `html-generated`, `local-asset`, `website-capture`), `purpose`, `prompt`, `placement`, `trackIndex`, and `avoid` (`caption`, `digital-human`, `key-ui`).
4. Prefer existing project assets and HTML/CSS/GSAP/chart scenes when they explain the point better. Use Chanjing AI Creation for generated backgrounds, source footage, transition clips, product display clips, or abstract visuals.
5. For AI B-roll, fetch models with `framevideo chanjing ai-models ...`, create metadata for each planned clip, run the idempotency checks above, submit only missing ready tasks, then use short sync runs to poll/download outputs to `assets/ai-creation/images/` or `assets/ai-creation/videos/` before use.
6. Insert B-roll as muted visual clips unless sound is explicitly required. Use empty tracks and avoid same-track overlap. Typical tracks: background `0`, B-roll `1-3`, text/cards `4-6`, captions `8`, digital human/A-roll `10`, narration `11`.
7. Adapt the visual with crop, scale, mask, darken, or blur as needed. Do not cover subtitles, presenter face/body, commands, or key product UI.
8. Validate: no remote `output_url`, local asset exists, no same-track overlap, captions are readable, digital human stays in frame, and no large blank areas. Run `npx framevideo lint` and `npx framevideo validate`; use `inspect` or `snapshot` for visual-heavy edits.
