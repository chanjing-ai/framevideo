# Chanjing AIGC Handoff

将 `Shot` 转换为蝉镜 AI Creation 可提交的任务计划。这个文件只定义从 AI production 到蝉镜 AIGC 的交接格式；提交、轮询、下载和回填规则见 `../../chanjing-digital-human/references/ai-creation.md`。

## 使用时机

读取本文件，当用户要求：

- 从创意、剧本或分镜继续到真实生图/生视频。
- 使用蝉镜 AIGC 生成 B-roll、背景、角色参考图、场景图、产品展示片段或抽象视觉素材。
- 把 `Shot` 级视频任务变成可提交的 Chanjing AI Creation 计划。
- 生成资产后继续回填 FrameVideo composition。

## 总原则

- 只适配蝉镜 AI Creation，不设计多模型抽象。
- `framevideo-ai-production` 负责叙事、Shot、连续性和生成意图。
- Chanjing AI Creation 负责模型列表、模型字段、计费任务、轮询、下载。
- 不硬编码 `model_id`、清晰度、比例、时长、参考图字段或模型私有参数。
- 模型列表和字段来自当前 FrameVideo CLI 或活跃 preview server。
- 所有远程输出必须下载为本地资产后才能写入 composition。

## 输入

优先接收标准 `Shot` 交接格式：

```text
Shot编号：
包含Clip：
总时长：
段落目标 / 微事件：
主视觉中心：
核心行为目标：
核心信息任务：
情绪张力：
场景与时间：
连续性约束：
必须保留的台词：
关键动作链：
镜头语言建议：
光影/滤镜建议：
动态场景/环境特效：
参考资产：
AI视频可生成性风险：
不并入前后段的理由：
```

如果输入仍是 Script 或 Clip，先回到对应 reference 生成 Clip / Shot，不要直接提交 AIGC。

## 输出：Chanjing AIGC Plan

每个需要生成的资产输出一个任务计划：

```yaml
id: shot-001
label: "S01 巷道对峙 B-roll"
source_shot: "Shot编号"
source_clips: ["Clip编号"]
adapter: chanjing-aigc
creation_type: 4
asset_kind: video
purpose: "主视觉 B-roll / 背景 / 参考图 / 转场素材 / 产品展示 / 抽象视觉"
start: 0
duration: 8
prompt: |
  面向蝉镜 AI Creation 的生成提示词。
negative:
  - 不要生成字幕
  - 不要改变人物身份
  - 不要改写台词
references:
  - id: "<<ref-character-01>>"
    role: character
    requirement: "保持人物身份、服装、发型和关键道具"
model_selection:
  command: "npx framevideo chanjing ai-models video --project <project> --compact"
  requirement: "从当前账号可用模型中选择支持本任务 reference/duration/aspect/quality 字段的模型"
payload_policy:
  source: "selected_model.params_config.fields"
  do_not_hardcode_fields: true
idempotency:
  metadata_path: ".framevideo/ai-creation-tasks/shot-001.json"
  fingerprint_inputs:
    - creation_type
    - selected model id/code/name
    - prompt
    - all user-controlled payload fields
    - reference image/resource ids or urls
output:
  local_dir: "assets/ai-creation/videos/"
  composition_track: 2
  avoid:
    - captions
    - digital-human-face
    - key-ui
risk:
  - "手部精细动作"
  - "低光人物一致性"
```

`creation_type` 使用蝉镜 AI Creation 约定：

- `3`: image
- `4`: video

## 任务类型选择

### 选择 image

用于：

- 角色参考图、场景概念图、产品静物图、背景板。
- 后续图生视频的首帧或参考图。
- FrameVideo 中可通过 GSAP/裁切/视差产生运动的静态素材。

输出目录：`assets/ai-creation/images/`

### 选择 video

用于：

- B-roll 片段。
- 环境动态，如雨、烟、霓虹、人群、抽象动效。
- 产品展示、转场素材、情绪镜头。
- 需要真实动态而不是 HTML/CSS 更合适的镜头。

输出目录：`assets/ai-creation/videos/`

### 优先不用 AIGC 的情况

如果画面是数据、UI、图表、文字解释、流程图、代码、软件操作或品牌排版，优先用 FrameVideo HTML/CSS/GSAP composition，而不是生成视频素材。AIGC 用于补充画面质感和 B-roll，不替代清晰的信息表达。

## Prompt 规则

从 Shot 字段合成 prompt：

- 主体来自 `主视觉中心`、`连续性约束`、`参考资产`。
- 动作来自 `核心行为目标` 和 `关键动作链`。
- 场景来自 `场景与时间`。
- 镜头来自 `镜头语言建议`。
- 光影来自 `光影/滤镜建议`。
- 环境动态来自 `动态场景/环境特效`。
- 台词只用于理解表演和节奏。除非模型任务明确支持口型/对白，否则不要要求模型准确生成口型。

必须保留：

- 原人物名、身份、关系。
- 参考资产标签原文。
- 连续性约束。
- 禁止字幕。

避免：

- 把多个 Shot 合成一个 AIGC 任务。
- 把大量台词塞进视频模型 prompt。
- 要求模型完成复杂手部细节、跨空间转场、多人复杂调度和强特效的混合任务。
- 远程 URL 直接进入 composition。

## 生成风险到计划的处理

- `手部精细动作`: 降低手部动作权重，拆成更短任务，或改用近景之外的构图。
- `多人调度`: 明确主视觉中心，减少同框人物，必要时拆分。
- `强动作`: 保留起势和结果，避免要求长链条连续动作。
- `复杂特效`: 单独生成特效背景或转场素材，不和关键表演混在一个任务。
- `空间转场`: 拆成转场前后两个任务，中间用 FrameVideo transition 连接。
- `大量台词`: 使用 FrameVideo TTS/字幕承载台词，AIGC 只生成视觉。

## 提交与同步交接

生成 Chanjing AIGC Plan 后，按以下顺序交给蝉镜 AI Creation：

1. 读取 `../../chanjing-digital-human/references/ai-creation.md`。
2. 获取模型列表：

```bash
npx framevideo chanjing ai-models image --project <project> --compact
npx framevideo chanjing ai-models video --project <project> --compact
```

3. 根据选中模型的 `params_config.fields` 写 payload。
4. 为每个计划创建 `.framevideo/ai-creation-tasks/*.json` 元数据。
5. 执行本地和远端幂等检查。
6. 仅提交缺失任务。
7. 使用短同步轮询下载结果。
8. 本地资产存在后，回到 FrameVideo composition。

## Composition 回填

视频资产使用：

```html
<video
  class="clip broll"
  src="assets/ai-creation/videos/shot-001.mp4"
  data-start="0"
  data-duration="8"
  data-track-index="2"
  muted
  playsinline
></video>
```

图片资产使用：

```html
<img
  class="clip broll-image"
  src="assets/ai-creation/images/shot-001.png"
  data-start="0"
  data-duration="5"
  data-track-index="2"
  alt=""
/>
```

回填后继续使用 `framevideo` skill：

- 为图片/视频加裁切、缩放、遮罩、暗角、scrim 或色彩适配。
- 保证 captions、数字人、CTA、关键 UI 不被遮挡。
- 运行 `npx framevideo lint`、`npx framevideo validate`、`npx framevideo inspect`。

## 自检

输出或提交前检查：

1. 每个 AIGC 任务是否只服务一个清晰 Shot 或素材目标。
2. 是否没有硬编码模型字段，而是等待模型列表和 `params_config.fields`。
3. 是否为每个任务定义了幂等 metadata path 和 fingerprint 输入。
4. 是否明确输出目录为本地 `assets/ai-creation/images/` 或 `assets/ai-creation/videos/`。
5. 是否没有把远程 URL 写入 composition。
6. 是否把不适合 AIGC 的信息表达任务留给 FrameVideo HTML/CSS/GSAP。
