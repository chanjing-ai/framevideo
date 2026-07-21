---
name: framevideo-media
description: Asset preprocessing for FrameVideo compositions — text-to-speech narration, Chanjing background music and sound effect downloads, audio/video transcription (Whisper), and background removal for transparent overlays (u2net). Use when generating voiceover from text, downloading BGM or SFX, transcribing speech for captions, removing the background from a video or image to use as a transparent overlay, choosing a TTS voice or whisper model, or chaining these (TTS → transcribe → captions).
---

# FrameVideo Media Preprocessing

## When To Use

Use this skill for:

- **Text-to-speech (TTS)** — generate voiceover narration from text (local Kokoro)
- **Background music (BGM)** — download music tracks from Chanjing platform
- **Sound effects (SFX)** — download short audio cues (clicks, whooshes, hits)
- **Transcription** — convert audio/video to timestamped captions (Whisper)
- **Background removal** — create transparent overlays from video/images (u2net)
- **Audio workflows** — chain commands (TTS → transcribe → captions)

## Do NOT Use

Avoid this skill for:

- **Digital human videos** — use `chanjing-digital-human`
- **Chanjing OAuth setup** — use `chanjing-auth`
- **SSML pronunciation control** — use `framevideo-voiceover-ssml`
- **Composition HTML** — use `framevideo`
- **Playing audio in compositions** — use `framevideo` (this skill only generates assets)

---

## Quick Start

Generate narration and captions in 3 steps:

```bash
# 1. Generate TTS audio
npx framevideo tts "Hello from FrameVideo" --output assets/narration.wav

# 2. Transcribe to get timestamps
npx framevideo transcribe assets/narration.wav --output assets/transcript.json

# 3. Reference in composition
# In index.html:
# <audio data-src="assets/narration.wav" data-start="0" data-track-index="10"></audio>
```

For background music:

```bash
# 1. Authenticate (one-time)
npx framevideo auth login

# 2. Browse music
npx framevideo chanjing music list --category <id> --compact

# 3. Download
npx framevideo chanjing music download --id <music-id> --output assets/music/bg.mp3
```

---

## Background Music vs Sound Effects

Use the two Chanjing audio paths intentionally:

- Background music (BGM): `chanjing music`, long track or chorus, `assets/music/`, default `data-volume="0.12"`, default `data-track-index="20"`.
- Sound effects (SFX): `chanjing sound-effect` or `chanjing sfx`, short event cues, `assets/sfx/`, default `data-volume="0.8"`, default `data-track-index="30"`.
- Both must be downloaded to local project assets before being referenced; never use remote Chanjing/OSS URLs directly in composition HTML.

## Background Music (`chanjing music`)

Use Chanjing OAuth-backed platform music when a user asks for background music, BGM, soundtrack, or chorus extraction from the Chanjing library. The command downloads the selected track to a local project asset and prints an `<audio>` snippet; never place remote Chanjing/OSS URLs directly in composition HTML.

```bash
npx framevideo auth status
npx framevideo chanjing music categories --json
npx framevideo chanjing music list --category <category-id> --compact
npx framevideo chanjing music download --id <music-id> --output assets/music/<name>.mp3 --volume 0.12
npx framevideo chanjing music download --id <music-id> --chorus --duration 10 --json
```

Defaults:

- Assets download under `assets/music/` when `--output` is omitted.
- `--output` supports `<name>` and `<id>` placeholders.
- Suggested background volume defaults to `0.12`; raise toward `0.22` only when there is no narration.
- `--chorus` calls the Chanjing chorus extraction endpoint and downloads the returned segment.

## Sound Effects (`chanjing sound-effect` / `chanjing sfx`)

Use Chanjing OAuth-backed platform sound effects when a user asks for UI clicks, whooshes, transitions, impact hits, notification sounds, or other short cues. The command downloads the selected effect to a local project asset and prints an `<audio>` snippet.

```bash
npx framevideo auth status
npx framevideo chanjing music categories --json     # shared audio category hints
npx framevideo chanjing sound-effect list --category <category-id> --compact
npx framevideo chanjing sound-effect download --id <effect-id> --output assets/sfx/<name>.mp3 --volume 0.8
npx framevideo chanjing sfx download --id <effect-id> --start 2.4 --json
```

Defaults:

- Assets download under `assets/sfx/` when `--output` is omitted.
- `--output` supports `<name>` and `<id>` placeholders.
- Suggested SFX volume defaults to `0.8`; use `0.6-1` depending on the cue and overall mix.
- SFX does not support chorus extraction.

## Text-to-Speech (`tts`)

Generate speech audio locally with Kokoro-82M. No API key.

### Chanjing-Backed Voice Requests

If the user asks for Chanjing voice generation, platform voices, digital-human narration, or any Chanjing-backed speech asset, invoke the `chanjing-digital-human` skill instead of defaulting to local Kokoro TTS. Only use local `npx framevideo tts` when the user explicitly wants offline speech or accepts a placeholder while Chanjing auth is unavailable.

```bash
npx framevideo tts "Text here" --voice af_nova --output narration.wav
npx framevideo tts script.txt --voice bf_emma --output narration.wav
npx framevideo tts --list                       # all 54 voices
```

### Voice Selection

Match voice to content. Default is `af_heart`.

| Content type      | Voice                 | Why                           |
| ----------------- | --------------------- | ----------------------------- |
| Product demo      | `af_heart`/`af_nova`  | Warm, professional            |
| Tutorial / how-to | `am_adam`/`bf_emma`   | Neutral, easy to follow       |
| Marketing / promo | `af_sky`/`am_michael` | Energetic or authoritative    |
| Documentation     | `bf_emma`/`bm_george` | Clear British English, formal |
| Casual / social   | `af_heart`/`af_sky`   | Approachable, natural         |

### Multilingual

Voice IDs encode language in the first letter: `a`=American English, `b`=British English, `e`=Spanish, `f`=French, `h`=Hindi, `i`=Italian, `j`=Japanese, `p`=Brazilian Portuguese, `z`=Mandarin. The CLI auto-detects the phonemizer locale from the prefix — no `--lang` needed when the voice matches the text.

```bash
npx framevideo tts "La reunión empieza a las nueve" --voice ef_dora --output es.wav
npx framevideo tts "今日はいい天気ですね" --voice jf_alpha --output ja.wav
```

Use `--lang` only to override auto-detection (stylized accents). Valid codes: `en-us`, `en-gb`, `es`, `fr-fr`, `hi`, `it`, `pt-br`, `ja`, `zh`. Non-English phonemization requires `espeak-ng` system-wide (`brew install espeak-ng` / `apt-get install espeak-ng`).

### Speed

- `0.7-0.8` — tutorial, complex content, accessibility
- `1.0` — natural pace (default)
- `1.1-1.2` — intros, transitions, upbeat content
- `1.5+` — rarely appropriate; test carefully

### Long Scripts

For more than a few paragraphs, write to a `.txt` file and pass the path. Inputs over ~5 minutes of speech may benefit from splitting into segments.

### Requirements

Python 3.8+ with `kokoro-onnx` and `soundfile` (`pip install kokoro-onnx soundfile`). Model downloads on first use (~311 MB + ~27 MB voices, cached in `~/.cache/framevideo/tts/`).

## Transcription (`transcribe`)

Produce a normalized `transcript.json` with word-level timestamps.

```bash
npx framevideo transcribe audio.mp3
npx framevideo transcribe video.mp4 --model small --language es
npx framevideo transcribe subtitles.srt          # import existing
npx framevideo transcribe subtitles.vtt
npx framevideo transcribe openai-response.json
```

### Language Rule (Non-Negotiable)

**Never use `.en` models unless the user explicitly states the audio is English.** `.en` models (`small.en`, `medium.en`) **translate** non-English audio into English instead of transcribing it. This silently destroys the original language.

1. Language known and non-English → `--model small --language <code>` (no `.en` suffix)
2. Language known and English → `--model small.en`
3. Language unknown → `--model small` (no `.en`, no `--language`) — whisper auto-detects

**Default model is `small`, not `small.en`.**

### Model Sizes

| Model      | Size   | Speed    | When to use                           |
| ---------- | ------ | -------- | ------------------------------------- |
| `tiny`     | 75 MB  | Fastest  | Quick previews, testing pipeline      |
| `base`     | 142 MB | Fast     | Short clips, clear audio              |
| `small`    | 466 MB | Moderate | **Default** — most content            |
| `medium`   | 1.5 GB | Slow     | Important content, noisy audio, music |
| `large-v3` | 3.1 GB | Slowest  | Production quality                    |

Music with vocals: start at `medium` minimum; produced tracks often need manual SRT/VTT import. For caption-quality checks (mandatory after every transcription), the cleaning JS, retry rules, and the OpenAI/Groq API import path, see [framevideo/references/transcript-guide.md](../framevideo/references/transcript-guide.md).

### Output Shape

Compositions consume a flat array of word objects. The `id` field (`w0`, `w1`, ...) is added during normalization for stable references in caption overrides; it's optional for backwards compatibility.

```json
[
  { "id": "w0", "text": "Hello", "start": 0.0, "end": 0.5 },
  { "id": "w1", "text": "world.", "start": 0.6, "end": 1.2 }
]
```

## Background Removal (`remove-background`)

Remove the background from a video or image so the subject (typically a person — avatar, presenter, talking head) sits as a transparent overlay in a composition.

```bash
npx framevideo remove-background subject.mp4 -o transparent.webm  # default: VP9 alpha WebM
npx framevideo remove-background subject.mp4 -o transparent.mov   # ProRes 4444 (editing)
npx framevideo remove-background portrait.jpg -o cutout.png       # single-image cutout
npx framevideo remove-background subject.mp4 -o subject.webm \
  --background-output plate.webm                                   # both layers in one pass
npx framevideo remove-background subject.mp4 -o transparent.webm --device cpu
npx framevideo remove-background --info                           # detected providers
```

Uses `u2net_human_seg` (MIT). First run downloads ~168 MB of weights to `~/.cache/framevideo/background-removal/models/`.

### Layer separation (`--background-output`)

Pass `--background-output` (or `-b`) to emit a **second** transparent video alongside the cutout: same source RGB, alpha is `255 − mask` instead of `mask`. The cutout is the subject with a transparent background; the plate is the original surroundings with a transparent hole where the subject was.

| File                             | Alpha is…                                                 | Use it for                                                      |
| -------------------------------- | --------------------------------------------------------- | --------------------------------------------------------------- |
| `-o subject.webm`                | The mask — subject opaque, background transparent         | Foreground layer, place on top                                  |
| `--background-output plate.webm` | Inverse — surroundings opaque, subject region transparent | Bottom layer; put text or graphics between this and the subject |

Both outputs share the same `--quality` preset and run from a single inference pass — encode cost roughly doubles, segmentation cost stays the same. Only valid for video inputs and `.webm`/`.mov` outputs.

**Hole-cut plate, not an inpainted clean plate.** The subject region in `plate.webm` is fully transparent — composite something opaque under it to fill the hole. The single test for whether `--background-output` is the right tool: _will anything ever be visible through the subject's silhouette where the subject used to be?_

| Use case                                                                            | Right tool                                                                         |
| ----------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| Text/graphics between the cutout and the plate (this command's reason for existing) | **Hole-cut** (`--background-output`)                                               |
| Subject onto an unrelated scene                                                     | Just `subject.webm`; ignore the plate                                              |
| Show the room _without_ the person, alone over no other content                     | **Clean plate** — needs an inpainter (LaMa, ProPainter, E2FGVI). Not this command. |
| Replace the subject with a different subject                                        | **Clean plate** — same as above                                                    |

If a user asks for "the room with the person removed" and intends to display it standalone, do **not** reach for `--background-output`. Tell them they need an inpainter.

Typical layered composition (the canonical hole-cut use case):

```html
<!-- z=1 the inverse-alpha plate fills everything except the subject region -->
<video
  src="plate.webm"
  data-start="0"
  data-duration="6"
  data-track-index="0"
  muted
  playsinline
></video>

<!-- z=2 graphics / text live between the two layers -->
<h1 id="headline" style="z-index:2; ...">MAKE IT IN FRAMEVIDEO</h1>

<!-- z=3 the cutout floats the subject back over the headline -->
<div class="cutout-wrap" style="position:absolute;inset:0;z-index:3">
  <video
    src="subject.webm"
    data-start="0"
    data-duration="6"
    data-track-index="1"
    muted
    playsinline
  ></video>
</div>
```

This is functionally equivalent to the text-behind-subject pattern below, but you don't need the original `presenter.mp4` in the project — the plate replaces it. Useful when you want to ship just the two transparent layers and let the user drop arbitrary content between them.

### Output Format

| Format                | When                                                          |
| --------------------- | ------------------------------------------------------------- |
| `.webm` (VP9 + alpha) | Default. Compositions play this directly via `<video>`.       |
| `.mov` (ProRes 4444)  | Editing in DaVinci/Premiere/FCP. Large files.                 |
| `.png`                | Single-image cutout (still subject, layered over a backdrop). |

Chrome decodes VP9 alpha natively, so the `.webm` plugs into a composition like any other muted-autoplay video — see the `framevideo` skill for the `<video>` track conventions.

### Quality presets

`--quality fast|balanced|best` controls only the VP9 encoder's CRF — segmentation quality is fixed.

| Preset     | CRF | When                                                  |
| ---------- | --- | ----------------------------------------------------- |
| `fast`     | 30  | Iterating, smaller file, looser color match           |
| `balanced` | 18  | Default. Visually identical for most uses             |
| `best`     | 12  | Master / final delivery. Largest file, tightest match |

### Compositing patterns — pick the right one

The cutout webm is a **re-encoded copy** of the source mp4's RGB. That choice has consequences depending on what you put behind it:

| Pattern                                                  | What's behind the cutout                   | Result                                                                                                                                                                                                                            |
| -------------------------------------------------------- | ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Cutout over a different scene** (most common)          | Static image, gradient, or unrelated video | Looks great. The cutout's RGB is the only source of the subject — no doubling, no edge halo. This is what `remove-background` is built for.                                                                                       |
| **Cutout over its own source mp4** (text-behind-subject) | Same mp4 the cutout was generated from     | Two RGB sources for the same person. At default `--quality balanced` (crf 18) the doubling is barely visible; at `--quality fast` (crf 30) you'll see a faint color shift / edge halo. Use `--quality best` (crf 12) for masters. |
| **Cutout over a _different_ take of the same person**    | Footage of the same subject                | Will look like two separate people overlapping. Don't do this.                                                                                                                                                                    |

**Text-behind-subject** (headline behind a presenter):

```html
<video
  src="presenter.mp4"
  id="bg"
  data-start="0"
  data-duration="6"
  data-track-index="0"
  muted
  playsinline
></video>
<h1 id="headline" style="z-index:2; ...">MAKE IT IN FRAMEVIDEO</h1>
<div class="cutout-wrap" style="position:absolute;inset:0;z-index:3;opacity:0">
  <video
    src="presenter.webm"
    data-start="0"
    data-duration="6"
    data-track-index="1"
    muted
    playsinline
  ></video>
</div>
```

Two key rules:

1. **Wrap the cutout video in a non-timed `<div>`** and animate the wrapper's opacity, not the video element's. The framework forces opacity:1 on active clips (any element with `data-start`/`data-duration`), so animating the video's opacity directly is silently overridden. The wrapper has no `data-*` attributes, so it's owned by your CSS/GSAP.
2. **Both videos use `data-start="0"` and `data-media-start="0"`** so the framework decodes them in sync from t=0. Late-mounting the cutout (`data-start=3.3`) introduces a seek + warm-up that lands a frame off the base mp4 — visible as one frame of misalignment at the cut.

Then GSAP-flip the wrapper opacity at the cut: `tl.set(cutoutWrap, { opacity: 1 }, 3.3)`.

## TTS → Transcribe → Captions

When there's no pre-recorded voiceover, generate one and transcribe it back to get word-level timestamps for captions:

```bash
npx framevideo tts script.txt --voice af_heart --output narration.wav
npx framevideo transcribe narration.wav   # → transcript.json
```

Whisper extracts precise word boundaries from the generated audio, so caption timing matches delivery without hand-tuning.

---

## Validation

After preprocessing assets:

```bash
# Verify audio files
npx framevideo inspect   # Check audio levels and duration

# Test in preview
npx framevideo preview   # Verify audio plays correctly in composition
```

**Manual checks:**

1. **File paths** — assets saved to correct directory (`assets/music/`, `assets/sfx/`, etc.)
2. **Audio levels** — BGM at 0.12-0.22, SFX at 0.6-1.0, narration at default
3. **Transcription accuracy** — review `transcript.json` word boundaries
4. **Background removal quality** — check transparent video for edge artifacts
5. **Integration** — verify asset referenced correctly in composition HTML

---

## Integration

This skill produces assets consumed by:
- **framevideo** — reference audio/video in composition HTML
- **framevideo-voiceover-ssml** — apply SSML markup before TTS generation
- **chanjing-digital-human** — alternative to local TTS for voiceover
- **website-to-framevideo** — Step 4 uses this skill for audio generation

---

## Credits And References

- Kokoro TTS: https://github.com/hexgrad/kokoro
- Whisper transcription: https://github.com/openai/whisper
- u2net background removal: https://github.com/xuebinqin/U-2-Net
- Chanjing platform: https://www.chanjing.cc
