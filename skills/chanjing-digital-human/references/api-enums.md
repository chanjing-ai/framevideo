# API Enums and Polling

## `digital_person_type`

| Value | Meaning |
| ----- | ------- |
| `0` | unknown / 未知 |
| `5` | custom LoRA digital human / 定制 lora 数字人 |
| `8` | real-person digital human / 真人数字人 |
| `9` | text-generated digital human / 文生数字人 |
| `10` | handheld-product digital human / 手持商品数字人 |
| `11` | creation-tool digital human / 创作工具数字人 |

## Layout and element enums

- **Direction:** `horizontal`, `vertical`
- **Canvas defaults:** horizontal `1920×1080`; vertical `1080×1920`
- **Background type:** `color`, `image`, `video`
- **Track/element type:** `person`, `pic`, `video`, `text`, `background` (plus older/internal `material`, `bubble`, `picture`)
- **Element source hint:** `common`, `custom`, `text`, `user` — metadata only; backend validates by id
- **Figure type:** `whole_body`, `sit_body`, `circle_view`, `origin` — user photo/text figures commonly use `origin`
- **Mouth mode:** `256`, `512`, `768` — backend default `256`
- **Backway:** `1` normal forward; `2` `normal_reverse` 正反播 — talking-head drafts default to `2`
- **Text photo gender:** `Female`, `Male`
- **Text photo aspect ratio:** `0` vertical 9:16; `1` horizontal 16:9

## Project status

`1` draft · `2` mobile draft · `10` submitted · `20` in production · `30` finished · `32` mobile export success · `40` error · `41` timeout · `42` mobile export failed · `43` AI film workspace build failed · `60` banned · `999` deleted · `9999` backstage deleted · `1000` soft expired · `1001` hard expired

## Submit task type

`make` (default) · `preview` · `montage` · `ip_montage` · `template`

## Task status (polling)

| Status | Meaning | Action |
| ------ | ------- | ------ |
| `10` | submitted | keep polling |
| `20` | in production | keep polling |
| `30` | finished | read `video_url`, download asset |
| `40` | error | stop, report `msg` |
| `41` | timeout | stop, report timeout |
| `50` | canceled | stop |
| `1100` | security manual review | keep polling or report waiting state |
| `1101` | security rejected | stop, report rejection |
| `999` | soft deleted | stop |

Typical synthesis takes 1–5 minutes. Poll every 2s, up to ~6 minutes (180 attempts).

```ts
const TERMINAL_FAIL = new Set([40, 41, 50, 1101, 999]);
const TERMINAL_OK = 30;

for (let i = 0; i < 180; i++) {
  const result = await client.getDigitalHumanVideo(taskId);
  if (result.status === TERMINAL_OK && result.video_url) return result;
  if (result.status !== undefined && TERMINAL_FAIL.has(result.status)) {
    throw new Error(result.msg ?? `Task ${taskId} failed with status ${result.status}`);
  }
  if (result.msg && /fail|error|失败/i.test(result.msg)) throw new Error(result.msg);
  await new Promise((r) => setTimeout(r, 2000));
}
throw new Error(`Timed out waiting for task ${taskId}`);
```
