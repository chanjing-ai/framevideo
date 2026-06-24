# Studio Routes and Plugin Client

## Studio routes

When the Studio server is running, prefer these project routes (implemented in `packages/cli/src/server/studioServer.ts`):

### Auth

```text
GET    /api/projects/:id/chanjing/auth/status
POST   /api/projects/:id/chanjing/auth/login
DELETE /api/projects/:id/chanjing/auth
```

`POST .../chanjing/auth/login` starts OAuth browser login. It must not accept or display `app_id` / `secret_key`.

### Digital humans

```text
GET  /api/projects/:id/digital-humans/common?tagIds=1&tagIds=2
GET  /api/projects/:id/digital-humans/tags
POST /api/projects/:id/digital-humans/custom
POST /api/projects/:id/digital-humans/sync
POST /api/projects/:id/digital-humans/text
POST /api/projects/:id/digital-humans/text/save
GET  /api/projects/:id/digital-humans/text/photo/status
```

`/digital-humans/generate` returns 501 by design (compatibility stub).

### Voices

```text
GET  /api/projects/:id/tts/voices?tagIds=1&tagIds=2
GET  /api/projects/:id/tts/voices/tags
POST /api/projects/:id/tts/voices/sync
```

`/tts` returns 501 by design (compatibility stub).

## Tag filtering (agent selection only)

`tagIds` only applies to public people and public voices:

- Public digital humans: `GET .../digital-humans/tags`, then `GET .../digital-humans/common?tagIds=...`
- Public voices: `GET .../tts/voices/tags`, then `GET .../tts/voices?tagIds=...`
- Custom digital humans and custom voices do not support tag filtering.

Use tag ids returned by the tag dictionary APIs. Multiple tag ids follow the website's existing narrowing semantics.

## Plugin client (`ChanjingOpenApiClient`)

Prefer `ChanjingOpenApiClient` from `packages/cli/src/tts/chanjingOpenapi.ts` over hand-written fetch for **save / submit / poll** synthesis. Despite the legacy name, it targets the plugin API with OAuth Bearer tokens.

| Method | Plugin path |
| ------ | ----------- |
| `listCommonDigitalPersons({ page, size, tagIds })` | `/digital_humans/common/list` |
| `listCommonDigitalPersonTags()` | `/digital_humans/common/tags` |
| `listCustomisedPersons({ page, size })` | `/digital_humans/custom/list` |
| `listCommonAudio({ page, size, tagIds })` | `/voices/common/list` |
| `listCommonVoiceTags()` | `/voices/common/tags` |
| `listCustomisedAudio({ page, size })` | `/voices/custom/list` |
| `saveWebsiteProject(payload)` | `/projects/save` |
| `submitWebsiteVideo({ projectId, ... })` | `/videos/submit` |
| `getDigitalHumanVideo(taskId)` | `/videos/status?task_id=...` |

Paths are relative to the plugin base URL.

- Default base: `http://localhost:8012/plugin/v1/chanjing` (local plugin debugging)
- Override: `CHANJING_PLUGIN_BASE_URL`, or `CHANJING_API_URL` (gets `/plugin/v1/chanjing` appended)

### Unsupported / creation-only

- `createDigitalHumanVideo(options)` — compatibility stub, returns 501. Do not use for new work.
- `listTextGeneratedDigitalPersons()` and text-photo/text-motion APIs — creation helpers only. When the **agent** picks a person, do not use ids from these until the person appears in `/digital-humans/custom`. When the **user** supplies a `person_id`, use it directly.

Do not pass a digital-human id as a website `projectId`.
