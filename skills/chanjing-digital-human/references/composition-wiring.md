# Composition Wiring

Use local generated assets in FrameVideo compositions. Do not reference remote Chanjing URLs at render time.

## Digital-human video with audio

Keep the visual `<video>` muted, then add a separate `<audio>` clip using the same local file for voice timing. Do not add silence placeholders.

```html
<video
  class="clip"
  src="assets/digital-humans/presenter-intro.mp4"
  data-start="0"
  data-duration="8"
  data-track-index="10"
  muted
  playsinline
></video>
<audio
  class="clip"
  src="assets/digital-humans/presenter-intro.mp4"
  data-start="0"
  data-duration="8"
  data-track-index="11"
  data-volume="1"
></audio>
```

## AI B-roll video

AI B-roll is usually a muted visual layer.

```html
<video
  class="clip broll"
  src="assets/ai-creation/videos/broll-01.mp4"
  data-start="8"
  data-duration="4"
  data-track-index="2"
  muted
  playsinline
></video>
```

## AI image asset

Use stable dimensions and object-fit so generated image aspect ratios do not shift layout.

```html
<img
  class="clip broll-image"
  src="assets/ai-creation/images/concept-01.png"
  data-start="12"
  data-duration="5"
  data-track-index="2"
  alt=""
/>
```

## Caption and presenter safety

- Keep captions above background/B-roll and below presenter-only overlays when needed. Track `8` is the common caption track.
- Keep digital-human/A-roll on a high visual track such as `10`; narration/audio can use `11`.
- Do not cover presenter face/body, subtitles, commands, or key product UI.
- For styled captions, prefer synthesis timing fields when available; otherwise invoke `framevideo-media` transcribe on the downloaded audio.
