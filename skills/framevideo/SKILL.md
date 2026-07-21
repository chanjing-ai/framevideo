---
name: framevideo
description: Create video compositions, animations, title cards, overlays, captions, voiceovers, audio-reactive visuals, and scene transitions in FrameVideo HTML. Use when asked to build any HTML-based video content, add captions or subtitles synced to audio, generate text-to-speech narration, create audio-reactive animation (beat sync, glow, pulse driven by music), add animated text highlighting (marker sweeps, hand-drawn circles, burst lines, scribble, sketchout), or add transitions between scenes (crossfades, wipes, reveals, shader transitions). Covers composition authoring, timing, media, and the full video production workflow. For dev-loop CLI commands (init, lint, inspect, preview, render) see the framevideo-cli skill; for asset preprocessing commands (tts, transcribe, remove-background) see the framevideo-media skill; for Chanjing digital humans, OAuth, and website-project synthesis see the chanjing-digital-human skill.
---

# FrameVideo

HTML is the source of truth for video. A composition is an HTML file with `data-*` attributes for timing, a GSAP timeline for animation, and CSS for appearance. The framework handles clip visibility, media playback, and timeline sync.

## When To Use

Use this skill for **any FrameVideo composition task**. This is the largest skill (1100+ lines) — use the navigation below to find what you need quickly.

### 🚀 New to FrameVideo?
1. Read "Quick Start" (line 46) — minimal working example
2. Read "Core Concepts" (line 95) — data attributes, timing, timeline contract
3. Try the example, then come back for specific features

### 🎯 Looking for something specific?

**Layout & Structure:**
- Safe areas & margins → "Layout & Safe Areas" (line 180)
- Multi-scene compositions → "Scene Transitions" (line 450) **[MANDATORY for multi-scene]**
- Composition architecture → "Composition Structure" (line 120)

**Animation & Motion:**
- Which animation library? → "Animation Adapter Routing" (line 28)
- GSAP patterns → Load `gsap` skill (default choice)
- Audio-reactive animation → "Audio Reactive Visuals" (line 620)
- Custom effects → Animation adapter skills (gsap, animejs, waapi, etc.)

**Media & Assets:**
- Video/audio playback → "Video & Audio" (line 140)
- Captions & subtitles → "Captions" (line 580)
- Parametrized compositions → "Variables" (line 160)

**Workflow:**
- AI-driven production → "AI Production Route" (line 200)
- Quality checks → `framevideo-visual-qa` skill
- CLI commands → `framevideo-cli` skill

**Do NOT use for:**
- CLI commands (init, lint, preview, render) → `framevideo-cli` skill
- Asset preprocessing (TTS, transcribe, bg-removal) → `framevideo-media` skill
- Quality checks and validation → `framevideo-visual-qa` skill
- Chanjing digital humans → `chanjing-digital-human` skill

---

## Animation Adapter Routing

Default to `gsap` for most FrameVideo composition animation. Use a specific adapter skill only when the content or user request calls for it:

| Use | Adapter skill |
| --- | --- |
| Scene choreography, text/card motion, staggered timelines, most scripted animation | `gsap` |
| User requests Anime.js, or porting compact Anime.js DOM/SVG examples | `animejs` |
| Simple finite CSS keyframes, shimmer, glow, masks, and non-sequenced decoration | `css-animations` |
| Lightweight native `element.animate()` motion with no external library | `waapi` |
| Existing Lottie/dotLottie assets from design tools | `lottie` |
| Deterministic 3D scenes, GLTF, WebGL, camera moves, shader plates | `three` |
| WebGPU/TypeGPU shaders, particles, liquid glass, compute pipelines | `typegpu` |

Shared rule for every adapter: render-critical animation must be deterministic and seekable. Do not use wall-clock time, infinite loops, or async registration for timelines/instances.

---

## Quick Start

Minimal working composition:

```html
<!doctype html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>My Video</title>
  </head>
  <body>
    <div data-composition-id="main" data-width="1920" data-height="1080">
      <div id="scene-1" class="clip" data-start="0" data-duration="5" data-track-index="1">
        <div class="scene-content">
          <h1>Hello FrameVideo</h1>
        </div>
      </div>

      <style>
        [data-composition-id="main"] { background: #000; color: #fff; }
        .scene-content {
          display: flex;
          align-items: center;
          justify-content: center;
          width: 100%;
          height: 100%;
          padding: 120px 160px;
          box-sizing: border-box;
        }
        h1 { font-size: 120px; }
      </style>

      <script src="https://cdn.jsdelivr.net/npm/gsap@3.14.2/dist/gsap.min.js"></script>
      <script>
        window.__timelines = window.__timelines || {};
        const tl = gsap.timeline({ paused: true });
        tl.from("h1", { opacity: 0, y: 40, duration: 0.8, ease: "power3.out" }, 0.3);
        window.__timelines["main"] = tl;
      </script>
    </div>
  </body>
</html>
```

Next: Validate with `npx framevideo lint`, preview with `npx framevideo preview`.

---

## Section Index

This skill has 1100+ lines organized into the following major sections. Jump to the section you need:

**🎬 Core Concepts** (lines 95-200) — Read first if new to FrameVideo
- Data Attributes & Timing
- Composition Structure  
- Timeline Contract
- Video & Audio
- Variables (Parametrized Compositions)

**🎨 Layout & Design** (lines 180-300)
- Safe Areas & Margins
- Typography & Readability
- Color & Contrast
- Responsive Layouts

**✨ Animation & Motion** (lines 300-450)
- Animation Adapter Routing (see above)
- GSAP Integration (or load `gsap` skill for details)
- Timing & Easing
- Stagger Patterns

**🎞️ Scene Transitions** (lines 450-580) — **MANDATORY for multi-scene videos**
- Crossfades
- Wipes & Reveals
- Shader Transitions
- Scene Architecture

**📝 Captions & Subtitles** (lines 580-650)
- Caption Timing
- Styling & Positioning
- Transcript Integration

**🎵 Audio-Reactive Visuals** (lines 620-720)
- Beat Detection
- Amplitude-Based Animation
- Music Synchronization

**🔧 Advanced Patterns** (lines 720-900)
- HTML-in-Canvas Effects
- Sub-Compositions
- Dynamic Content
- Performance Optimization

**📚 References** (lines 900-1100)
- Examples Gallery
- Troubleshooting
- Best Practices
- API Reference

**Note:** Line numbers are approximate guides. Use your Read tool's offset parameter to jump to sections, or read the full file if you need comprehensive context.

---

## Core Concepts
- Video & Audio Elements
- Variables (Parametrized Compositions)

**Layout & Safe Areas** (expand when positioning elements):
- Layout Before Animation Principle
- Container Patterns
- Safe Area Rules
- Text Handling
- Common Layout Patterns

**Animation & Motion** (expand when adding motion):
- Animation Guardrails
- Entrance/Exit Patterns
- Stagger & Sequencing
- Scene Rhythm Templates
- Animation Conflicts

**Scene Transitions** (mandatory for multi-scene videos):
- 4 Non-Negotiable Rules
- Transition Implementation

**Advanced Features** (as needed):
- Captions & Subtitles
- Audio-Reactive Animation
- CSS Marker Highlighting
- Shader Transitions

**References** (deep dives):
- See "References" section at end for 15+ detailed guides

---

## Core Concepts

### Data Attributes & Timing

Every clip requires these attributes:

| Attribute          | Required | Values                                                 |
| ------------------ | -------- | ------------------------------------------------------ |
| `id`               | Yes      | Unique identifier                                      |
| `data-start`       | Yes      | Seconds or clip ID reference (`"el-1"`, `"intro + 2"`) |
| `data-duration`    | Yes*     | Seconds. *Optional for video/audio (uses media length) |
| `data-track-index` | Yes      | Integer. Same-track clips cannot overlap              |

Optional:
- `data-media-start` - Trim offset into source (seconds)
- `data-volume` - 0-1 (audio only, default 1)

**Important:** `data-track-index` does NOT control visual layering. Use CSS `z-index`.

### Composition Structure

**Root element** with required attributes:

```html
<div data-composition-id="main" data-width="1920" data-height="1080">
  <!-- content -->
</div>
```

**Standalone compositions** (main `index.html`): Put root `<div>` directly in `<body>`. Do NOT use `<template>`.

**Sub-compositions** (loaded via `data-composition-src`): MUST use `<template>` wrapper:

```html
<template id="my-comp-template">
  <div data-composition-id="my-comp" data-width="1920" data-height="1080">
    <!-- content -->
  </div>
</template>
```

### Timeline Contract

**5 mandatory rules:**

1. **Create paused:** `const tl = gsap.timeline({ paused: true });`
2. **Register timeline:** `window.__timelines["main"] = tl;` (key must match `data-composition-id`)
3. **Synchronous construction:** Never build timelines inside `async`, `setTimeout`, or Promises
4. **Duration from data attribute:** Use `data-duration`, not GSAP timeline length
5. **Framework auto-nests:** Don't manually add sub-composition timelines

### Video & Audio

**Video must be muted:**

```html
<video
  id="el-v"
  data-start="0"
  data-duration="30"
  data-track-index="0"
  src="video.mp4"
  muted
  playsinline
></video>
```

**Audio track separate:**

```html
<audio
  id="el-a"
  data-start="0"
  data-duration="30"
  data-track-index="2"
  src="video.mp4"
  data-volume="1"
></audio>
```

Use same source file. Framework controls playback - never call `.play()`.

### Variables (Parametrized Compositions)

**Three-step pattern:**

1. **Declare** on `<html>` root:
```html
<html data-composition-variables='[
  {"id":"title","type":"string","label":"Title","default":"Hello"}
]'>
```

2. **Read** in script:
```javascript
const { title } = window.__framevideo.getVariables();
document.getElementById("hero").textContent = title;
```

3. **Override** at render:
```bash
npx framevideo render --variables '{"title":"Q4 Report"}'
```

Variable types: `string`, `number`, `color`, `boolean`, `enum`.

---

<details>
<summary><h2>📐 Layout & Safe Areas (Click to Expand)</h2></summary>

## AI Production Route

When the user asks to go from an idea, script, plot, storyboard, reference image, or rough video concept to a generated video, invoke `framevideo-ai-production` as the front half of the workflow. That skill turns creative material into Clip, Shot, and Chanjing AIGC plans.

For real Chanjing AI image/video generation:

1. Use `framevideo-ai-production` to produce Shot-level Chanjing AIGC plans.
2. Use `chanjing-digital-human/references/ai-creation.md` for Chanjing AI Creation model discovery, idempotent submission, short sync polling, download, and local asset paths.
3. Only compose with local assets under `assets/ai-creation/images/` or `assets/ai-creation/videos/`.
4. Return here to build the FrameVideo HTML composition, audio, captions, transitions, QA, preview, and render.

Do not directly insert remote Chanjing output URLs into composition HTML.

## Approach

### Discovery (exploratory requests only)

For open-ended requests ("make me a product launch video", "create something for our brand") where the user hasn't committed to a direction, understand intent before picking colors:

- **Audience** — who watches this? Developers? Executives? General consumers?
- **Platform** — where does it play? Social (15s), website hero, product demo, internal?
- **Priority** — what matters most? Motion quality? Content accuracy? Brand fidelity? Speed?
- **Variations** — does the user want options, or a single best shot?

For specific requests ("add a title card", "fix the timing on scene 3"), skip discovery.

For exploratory requests, consider offering 2-3 variations that differ meaningfully — not just color swaps, but different pacing, energy levels, or structural approaches. One safe/expected, one ambitious. Don't mandate this — it's a tool available when appropriate.

### Step 1: Design system

If a design spec exists in the project, read it first. Look in precedence order: `frame.md` → `design.md` → `DESIGN.md` (`design.md` and `DESIGN.md` are different files on Linux — check both casings; `frame.md` is always lowercase, no `FRAME.md` variant). `frame.md` is the preferred spec for video/framevideo projects and wins if more than one exists; it uses the same format as `design.md`. It's the source of truth for brand colors, fonts, and constraints. Use its exact values — don't invent colors or substitute fonts. Any format works (YAML frontmatter, prose, tables — just extract the values).

If it names fonts you can't find locally (no `fonts/` directory with `.woff2` files, not a built-in font), warn the user before writing HTML: "the spec specifies [font name] but no font files found. Please add .woff2 files to `fonts/` or I'll fall back to [closest built-in alternative]."

If no `frame.md` or `design.md` exists, offer the user a choice:

1. **User named a style or mood?** → Read [visual-styles.md](./visual-styles.md) for the 8 named presets. Pick the closest match.
2. **Want to browse options visually?** → Run the design picker: read [references/design-picker.md](references/design-picker.md) for the full workflow. This serves a visual picker page. The user configures mood, palette, typography, and motion in the browser, then copies the generated design.md and pastes it back into the conversation.
3. **Want to skip and go fast?** → Ask: mood, light or dark, any brand colors/fonts? Then pick a palette from [house-style.md](./house-style.md).

**The design spec defines the brand. It does not define video composition rules.** Those come from [references/video-composition.md](references/video-composition.md) and [house-style.md](./house-style.md). Use brand colors at video-appropriate scale — not at web-UI opacity.

### Step 2: Prompt expansion

Always run on every composition (except single-scene pieces and trivial edits). This step grounds the user's intent against the design spec (`frame.md` or `design.md`) and `house-style.md` and produces a consistent intermediate that every downstream agent reads the same way.

Read [references/prompt-expansion.md](references/prompt-expansion.md) for the full process and output format.

### Step 3: Plan

Before writing HTML, think at a high level:

1. **What** — what should the viewer experience? Identify the narrative arc, key moments, and emotional beats.
2. **Structure** — how many compositions, which are sub-compositions vs inline, what tracks carry what (video, audio, overlays, captions).
3. **Rhythm** — declare your scene rhythm before implementing. Which scenes are quick hits, which are holds, where do shaders land, where does energy peak. Name the pattern: fast-fast-SLOW-fast-SHADER-hold. Read [references/beat-direction.md](references/beat-direction.md) for rhythm templates.
4. **Timing** — which clips drive the duration, where do transitions land, what's the pacing.
5. **Layout** — build the end-state first. See "Layout Before Animation" below.
6. **Animate** — then add motion using the rules below.

**Build what was asked.** A request for "a title card" is not a request for "a title card + 3 supporting scenes + ambient music + captions." Every scene, every element, every tween should earn its place. If additional scenes or elements would genuinely improve the piece, propose them — don't add them.

For small edits (fix a color, adjust timing, add one element), skip straight to the rules.

<HARD-GATE>
Before writing ANY composition HTML — verify you have a visual identity from Step 1. If you're reaching for `#333`, `#3b82f6`, or `Roboto`, you skipped it.
</HARD-GATE>

### Visual QA Skill

Use the `framevideo-visual-qa` skill when creating a new composition, making substantial layout/caption/asset/layering changes, fixing visual issues, or before declaring a composition visually ready. That skill owns reusable video QA rules for safe areas, text/background contrast, text overflow, unintentional overlap, and motion collisions. Keep project-specific brand identity in `frame.md`, `design.md`, or `DESIGN.md`; keep general visual QA method in the skill.

---

<details>
<summary><h2>📐 Layout & Safe Areas (Click to Expand)</h2></summary>

### Layout Before Animation Principle

**Build the end state first.** Position every element where it should be at its **most visible moment** - fully entered, correctly placed, not yet exiting. Write this as static HTML+CSS first. No GSAP yet.

**Why:** If you position elements at their animated start state (offscreen, scaled to 0, opacity 0) and tween them to where you think they should land, you're guessing. Overlaps are invisible until render. By building the end state first, you see and fix layout problems before adding motion.

**The process:**

1. **Identify the hero frame** - the moment when most elements are simultaneously visible
2. **Write static CSS** for that frame
3. **Add entrances with `gsap.from()`** - animate FROM offscreen/invisible TO the CSS position
4. **Add exits with `gsap.to()`** - animate TO offscreen/invisible FROM the CSS position

**Example:**

```css
/* Step 1-2: Build the readable end state */
.scene-content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  width: 100%;
  height: 100%;
  padding: 120px 160px;
  gap: 24px;
  box-sizing: border-box;
}
.title { font-size: 120px; }
.subtitle { font-size: 42px; }
```

```javascript
// Step 3: Animate INTO those positions
tl.from(".title", { y: 60, opacity: 0, duration: 0.6, ease: "power3.out" }, 0);
tl.from(".subtitle", { y: 40, opacity: 0, duration: 0.5, ease: "power3.out" }, 0.2);

// Step 4: Animate OUT from those positions
tl.to(".title", { y: -40, opacity: 0, duration: 0.4, ease: "power2.in" }, 3);
```

### Container Patterns

**✅ Full-Scene Container (Recommended):**

```css
.scene-content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  padding: 120px 160px; /* Safe area */
  gap: 24px;
  box-sizing: border-box;
}
```

**❌ Do NOT use absolute positioning for content:**

```css
/* WRONG - breaks on long text */
.scene-content {
  position: absolute;
  top: 200px;
  left: 160px;
  width: 1920px;
  height: 1080px;
}
```

Reserve `position: absolute` for decorative elements only (particles, glows, background shapes).

### Safe Area Rules

Keep key information inside the safe area:

- **Horizontal:** ~5% from left/right edges (~96px on 1920px width)
- **Vertical:** ~5% from top/bottom edges (~54px on 1080px height)
- **Bottom (captions/CTA):** ~10% from bottom (~108px)

**What must stay safe:**
- Critical text, headlines, body copy
- Logo (unless intentionally edge-anchored)
- CTA buttons and links
- Captions and subtitles
- Presenter faces
- Product UI screenshots
- Legal/price copy

**What can extend beyond:**
- Full-bleed backgrounds
- Decorative glows and particles
- Grain and texture overlays
- Transition effects

Mark intentional overflow: `<div class="grain" data-layout-allow-overflow>`

### Text Handling

**Natural wrapping:**

```css
.headline {
  max-width: 1200px; /* Wraps at this width */
  font-size: 96px;
  line-height: 1.1;
}
```

**Dynamic fitting:**

```javascript
const { title } = window.__framevideo.getVariables();
const fontSize = window.__framevideo.fitTextFontSize(title, {
  maxWidth: 1200,
  fontFamily: 'Inter',
  fontWeight: 700
});
document.getElementById('title').style.fontSize = fontSize + 'px';
```

**Avoid fixed-size text containers** - use elastic containers with padding instead.

### Common Layout Patterns

**Centered Title Card:**

```html
<div class="scene-content">
  <h1>Main Message</h1>
  <p class="subtitle">Supporting detail</p>
</div>

<style>
  .scene-content {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-align: center;
    width: 100%;
    height: 100%;
    padding: 120px 160px;
    gap: 32px;
    box-sizing: border-box;
  }
</style>
```

**Split Layout (Text + Visual):**

```html
<div class="scene-content">
  <div class="text-side">
    <h2>Product Feature</h2>
    <p>Explanation</p>
  </div>
  <div class="visual-side">
    <img src="assets/product.png" />
  </div>
</div>

<style>
  .scene-content {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 80px;
    align-items: center;
    width: 100%;
    height: 100%;
    padding: 120px 160px;
    box-sizing: border-box;
  }
</style>
```

</details>

---

<details>
<summary><h2>🎬 Animation & Motion (Click to Expand)</h2></summary>

### Animation Guardrails

**Timing offsets** - Never start first animation at t=0:

```javascript
// ❌ Wrong
tl.from(".title", { opacity: 0 }, 0);

// ✅ Right
tl.from(".title", { opacity: 0 }, 0.2); // 0.1-0.3s offset
```

**Vary easing curves** - Use at least 3 different eases per scene:

```javascript
tl.from("#el1", { opacity: 0, ease: "power3.out" }, 0.2);
tl.from("#el2", { opacity: 0, ease: "expo.out" }, 0.4);
tl.from("#el3", { opacity: 0, ease: "back.out(1.4)" }, 0.6);
```

Common eases: `power1.out`, `power2.out`, `power3.out`, `expo.out`, `back.out(1.7)`, `elastic.out(1, 0.3)`

**Don't repeat entrance patterns** - Each element should have unique direction or style:

```javascript
// ❌ Wrong - everything from top
tl.from("#title", { y: 60, opacity: 0 }, 0.2);
tl.from("#subtitle", { y: 60, opacity: 0 }, 0.5);

// ✅ Right - varied directions
tl.from("#title", { y: 60, opacity: 0 }, 0.2);
tl.from("#subtitle", { x: -30, opacity: 0 }, 0.5);
tl.from("#cta", { scale: 0.8, opacity: 0 }, 0.8);
```

**Performance properties** - Prefer GPU-accelerated:

- ✅ Fast: `opacity`, `x`, `y`, `scale`, `rotation`
- ❌ Slow: `width`, `height`, `top`, `left`, `margin`, `padding`, `font-size`

### Common Animation Patterns

**Stagger entrances:**

```javascript
// Stagger by time
tl.from(".item", { 
  y: 30, 
  opacity: 0, 
  duration: 0.5,
  stagger: 0.1,
  ease: "power2.out"
}, 0.5);

// Stagger from center
tl.from(".grid-item", {
  scale: 0.8,
  opacity: 0,
  duration: 0.4,
  stagger: { amount: 0.6, from: "center" },
  ease: "back.out(1.4)"
}, 1.0);
```

**Sequence vs Overlap:**

```javascript
// Sequence - one after another
tl.from("#el1", { opacity: 0, duration: 0.5 }, 0);
tl.from("#el2", { opacity: 0, duration: 0.5 }, ">"); // After previous ends

// Overlap - start before previous ends
tl.from("#el1", { opacity: 0, duration: 0.8 }, 0);
tl.from("#el2", { opacity: 0, duration: 0.5 }, "<0.4"); // 0.4s after el1 starts
```

**Label-based sequencing:**

```javascript
tl.addLabel("intro", 0);
tl.from("#title", { opacity: 0 }, "intro");
tl.from("#subtitle", { opacity: 0 }, "intro+=0.5");

tl.addLabel("content", 2.5);
tl.from("#feature1", { x: -40, opacity: 0 }, "content");
```

### Scene Rhythm Templates

Declare rhythm before implementing:

- **Fast-Fast-SLOW:** Quick intro beats → hold for message (2s, 3s, 5s)
- **Build-PEAK-Resolve:** Escalating energy → climax → settle (3s, 2s, 4s)
- **Even Pulse:** Consistent rhythm (all beats 3-4s, good for instructional)

See [references/beat-direction.md](references/beat-direction.md) for more templates.

### Animation Conflicts

Never animate the same property on the same element from multiple timelines:

```javascript
// ❌ Wrong - both animate opacity
timeline1.to("#el", { opacity: 0.5 });
timeline2.to("#el", { opacity: 1 }); // Conflict!

// ✅ Right - separate properties
timeline1.to("#el", { opacity: 0.5 });
timeline2.to("#el", { x: 100 }); // No conflict
```

</details>

---

## Scene Transitions (Non-Negotiable)

**Every multi-scene composition MUST follow these 4 rules:**

### Rule 1: Always Use Transitions
No jump cuts between scenes. Every scene change needs a transition.

### Rule 2: Always Use Entrance Animations
Every element in every scene animates IN via `gsap.from()`. No element may appear fully-formed.

### Rule 3: Never Use Exit Animations (Except Final Scene)
Do NOT animate elements out before a transition. The transition IS the exit. The outgoing scene's content MUST be fully visible when the transition starts.

### Rule 4: Final Scene Only
The last scene is the ONLY scene where `gsap.to(..., { opacity: 0 })` or exit animations are allowed.

**Example:**

```javascript
// Scene 1 (0-7s)
// ✅ Entrance animations only
tl.from("#s1-title", { y: 50, opacity: 0, duration: 0.7, ease: "power3.out" }, 0.3);
tl.from("#s1-subtitle", { y: 30, opacity: 0, duration: 0.5, ease: "power2.out" }, 0.6);
// ❌ NO exit animations - transition handles it

// Transition at 7s (see references/transitions.md)

// Scene 2 (8-15s)
// ✅ Entrance animations
tl.from("#s2-heading", { x: -40, opacity: 0, duration: 0.6, ease: "expo.out" }, 8.0);

// Final scene (15-20s)
// ✅ OK to fade out on final scene only
tl.from("#s3-cta", { scale: 0.9, opacity: 0, duration: 0.5 }, 15.2);
tl.to("#s3-cta", { opacity: 0, duration: 0.5, ease: "power2.in" }, 19.5);
```

See [references/transitions.md](references/transitions.md) for transition implementation.

---
## Layout Before Animation

```css
.scene-content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  width: 100%;
  height: 100%;
  padding: 120px 160px;
  gap: 24px;
  box-sizing: border-box;
}
.title {
  font-size: 120px;
}
.subtitle {
  font-size: 42px;
}
/* Container fills any scene size (1920x1080, 1080x1920, etc).
   Padding positions content. Flex + gap handles spacing. */
```

**WRONG — hardcoded dimensions and absolute positioning:**

```css
.scene-content {
  position: absolute;
  top: 200px;
  left: 160px;
  width: 1920px;
  height: 1080px;
  display: flex; /* ... */
}
```

```js
// Step 3: Animate INTO those positions
tl.from(".title", { y: 60, opacity: 0, duration: 0.6, ease: "power3.out" }, 0);
tl.from(".subtitle", { y: 40, opacity: 0, duration: 0.5, ease: "power3.out" }, 0.2);
tl.from(".logo", { scale: 0.8, opacity: 0, duration: 0.4, ease: "power2.out" }, 0.3);

// Step 4: Animate OUT from those positions
tl.to(".title", { y: -40, opacity: 0, duration: 0.4, ease: "power2.in" }, 3);
tl.to(".subtitle", { y: -30, opacity: 0, duration: 0.3, ease: "power2.in" }, 3.1);
tl.to(".logo", { scale: 0.9, opacity: 0, duration: 0.3, ease: "power2.in" }, 3.2);
```

### When elements share space across time

If element A exits before element B enters in the same area, both should have correct CSS positions for their respective hero frames. The timeline ordering guarantees they never visually coexist — but if you skip the layout step, you won't catch the case where they accidentally overlap due to a timing error.

### What counts as intentional overlap

Layered effects (glow behind text, shadow elements, background patterns) and z-stacked designs (card stacks, depth layers) are intentional. The layout step is about catching **unintentional** overlap — two headlines landing on top of each other, a stat covering a label, content bleeding off-frame.

## Data Attributes

### All Clips

| Attribute          | Required                          | Values                                                 |
| ------------------ | --------------------------------- | ------------------------------------------------------ |
| `id`               | Yes                               | Unique identifier                                      |
| `data-start`       | Yes                               | Seconds or clip ID reference (`"el-1"`, `"intro + 2"`) |
| `data-duration`    | Required for img/div/compositions | Seconds. Video/audio defaults to media duration.       |
| `data-track-index` | Yes                               | Integer. Same-track clips cannot overlap.              |
| `data-media-start` | No                                | Trim offset into source (seconds)                      |
| `data-volume`      | No                                | 0-1 (default 1)                                        |

`data-track-index` does **not** affect visual layering — use CSS `z-index`.

### Composition Clips

| Attribute                    | Required | Values                                                            |
| ---------------------------- | -------- | ----------------------------------------------------------------- |
| `data-composition-id`        | Yes      | Unique composition ID                                             |
| `data-start`                 | Yes      | Start time (root composition: use `"0"`)                          |
| `data-duration`              | Yes      | Takes precedence over GSAP timeline duration                      |
| `data-width` / `data-height` | Yes      | Pixel dimensions (1920x1080 or 1080x1920)                         |
| `data-composition-src`       | No       | Path to external HTML file                                        |
| `data-variable-values`       | No       | JSON object of per-instance variable overrides on a sub-comp host |

On the root `<html>` element:

| Attribute                    | Required | Values                                                                                                                         |
| ---------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `data-composition-variables` | No       | JSON array of declared variables (id/type/label/default) — drives Studio editing UI and provides defaults for `getVariables()` |

## Composition Structure

Sub-compositions loaded via `data-composition-src` use a `<template>` wrapper. **Standalone compositions (the main index.html) do NOT use `<template>`** — they put the `data-composition-id` div directly in `<body>`. Using `<template>` on a standalone file hides all content from the browser and breaks rendering.

Sub-composition structure:

```html
<template id="my-comp-template">
  <div data-composition-id="my-comp" data-width="1920" data-height="1080">
    <!-- content -->
    <style>
      [data-composition-id="my-comp"] {
        /* scoped styles */
      }
    </style>
    <script src="https://cdn.jsdelivr.net/npm/gsap@3.14.2/dist/gsap.min.js"></script>
    <script>
      window.__timelines = window.__timelines || {};
      const tl = gsap.timeline({ paused: true });
      // tweens...
      window.__timelines["my-comp"] = tl;
    </script>
  </div>
</template>
```

Load in root: `<div id="el-1" data-composition-id="my-comp" data-composition-src="compositions/my-comp.html" data-start="0" data-duration="10" data-track-index="1"></div>`

## Variables (Parametrized Compositions)

Render the same composition with different content — title, theme color, prices, captions — without editing the source HTML.

**Three-step pattern:**

1. **Declare** variables on the composition's `<html>` root with `data-composition-variables`. Each entry needs `id`, `type` (one of `string`, `number`, `color`, `boolean`, `enum`), `label`, and `default`. Enum entries also need `options: [{value, label}, ...]`.
2. **Read** the resolved values inside the composition's script with `window.__framevideo.getVariables()`. Returns the merged result of declared defaults + per-instance overrides + CLI overrides.
3. **Override** at render time with `npx framevideo render --variables '{...}'` (top-level) or with `data-variable-values='{...}'` on the host element (per-instance for sub-comps).

```html
<!doctype html>
<html
  data-composition-variables='[
  {"id":"title","type":"string","label":"Title","default":"Hello"},
  {"id":"theme","type":"enum","label":"Theme","default":"light","options":[
    {"value":"light","label":"Light"},
    {"value":"dark","label":"Dark"}
  ]}
]'
>
  <body>
    <div data-composition-id="root" data-width="1920" data-height="1080">
      <h1 id="hero" class="clip" data-start="0" data-duration="3"></h1>
      <script>
        const { title, theme } = window.__framevideo.getVariables();
        document.getElementById("hero").textContent = title;
        document.body.dataset.theme = theme;
      </script>
    </div>
  </body>
</html>
```

```bash
# Dev preview uses declared defaults
npx framevideo preview

# Render with overrides
npx framevideo render --variables '{"title":"Q4 Report","theme":"dark"}' --output q4.mp4

# Or from a JSON file
npx framevideo render --variables-file ./vars.json
```

**Sub-composition per-instance values:** the same `getVariables()` works inside sub-comps loaded via `data-composition-src`. Each host element passes its own values:

```html
<div
  data-composition-id="card-pro"
  data-composition-src="compositions/card.html"
  data-variable-values='{"title":"Pro","price":"$29"}'
></div>
<div
  data-composition-id="card-enterprise"
  data-composition-src="compositions/card.html"
  data-variable-values='{"title":"Enterprise","price":"Custom"}'
></div>
```

The runtime layers each host's `data-variable-values` over the sub-comp's declared defaults on a per-instance basis, so the same source can be embedded multiple times with different content.

**Rules of thumb:**

- Always provide a sensible `default` for every declared variable. Dev preview uses defaults — without them, the composition won't render correctly until `--variables` is provided.
- Read variables once at the top of the script (`const { title } = ...`), not inside frame loops or event handlers — `getVariables()` allocates a fresh object per call.
- Use `--strict-variables` in CI to fail fast on undeclared keys or type mismatches.
- Variable types are validated at render time. `string`, `number`, `boolean`, and `color` (hex string) check `typeof`; `enum` checks the value is in the declared `options`.

## Video and Audio

Video must be `muted playsinline`. Audio is always a separate `<audio>` element:

```html
<video
  id="el-v"
  data-start="0"
  data-duration="30"
  data-track-index="0"
  src="video.mp4"
  muted
  playsinline
></video>
<audio
  id="el-a"
  data-start="0"
  data-duration="30"
  data-track-index="2"
  src="video.mp4"
  data-volume="1"
></audio>
```

For background music from the Chanjing platform, use the CLI to download a local asset first:

```bash
npx framevideo chanjing music list --compact
npx framevideo chanjing music download --id <music-id> --chorus --duration 10 --json
```

Use the returned `htmlSnippet` or author the same shape manually. Keep BGM local under `assets/music/`; do not reference remote Chanjing/OSS URLs directly in composition HTML. Suggested BGM volume is `0.10-0.22`; use `data-volume="0.12"` when speech or digital-human narration is present.

For sound effects from the Chanjing platform, download the SFX asset first and place it at the event time:

```bash
npx framevideo chanjing sound-effect list --compact
npx framevideo chanjing sfx download --id <effect-id> --volume 0.8 --json
```

Keep SFX local under `assets/sfx/`. Use short, event-specific `<audio>` clips with `data-start` set to the exact cue time, `data-track-index="30"` or higher, and `data-volume="0.6-1"` depending on the mix.

## Timeline Contract

- All timelines start `{ paused: true }` — the player controls playback
- Register every timeline: `window.__timelines["<composition-id>"] = tl`
- Framework auto-nests sub-timelines — do NOT manually add them
- Duration comes from `data-duration`, not from GSAP timeline length
- Never create empty tweens to set duration

## Rules (Non-Negotiable)

**Deterministic:** No `Math.random()`, `Date.now()`, or time-based logic. Use a seeded PRNG if you need pseudo-random values (e.g. mulberry32).

**GSAP:** Only animate visual properties (`opacity`, `x`, `y`, `scale`, `rotation`, `color`, `backgroundColor`, `borderRadius`, transforms). Do NOT animate `visibility`, `display`, or call `video.play()`/`audio.play()`.

**Animation conflicts:** Never animate the same property on the same element from multiple timelines simultaneously.

**No `repeat: -1`:** Infinite-repeat timelines break the capture engine. Calculate the exact repeat count from composition duration: `repeat: Math.ceil(duration / cycleDuration) - 1`.

**Synchronous timeline construction:** Never build timelines inside `async`/`await`, `setTimeout`, or Promises. The capture engine reads `window.__timelines` synchronously after page load. Fonts are embedded by the compiler, so they're available immediately — no need to wait for font loading.

**Never do:**

1. Forget `window.__timelines` registration
2. Use video for audio — always muted video + separate `<audio>`
3. Nest video inside a timed div — use a non-timed wrapper
4. Use `data-layer` (use `data-track-index`) or `data-end` (use `data-duration`)
5. Animate video element dimensions — animate a wrapper div
6. Call play/pause/seek on media — framework owns playback
7. Create a top-level container without `data-composition-id`
8. Use `repeat: -1` on any timeline or tween — always finite repeats
9. Build timelines asynchronously (inside `async`, `setTimeout`, `Promise`)
10. Use `gsap.set()` on clip elements from later scenes — they don't exist in the DOM at page load. Use `tl.set(selector, vars, timePosition)` inside the timeline at or after the clip's `data-start` time instead.
11. Use `<br>` in content text — forced line breaks don't account for actual rendered font width. Text that wraps naturally + a `<br>` produces an extra unwanted break, causing overlap. Let text wrap via `max-width` instead. Exception: short display titles where each word is deliberately on its own line (e.g., "THE\nIMMORTAL\nGAME" at 130px).

## Scene Transitions (Non-Negotiable)

Every multi-scene composition MUST follow ALL of these rules. Violating any one of them is a broken composition.

1. **ALWAYS use transitions between scenes.** No jump cuts. No exceptions.
2. **ALWAYS use entrance animations on every scene.** Every element animates IN via `gsap.from()`. No element may appear fully-formed. If a scene has 5 elements, it needs 5 entrance tweens.
3. **NEVER use exit animations** except on the final scene. This means: NO `gsap.to()` that animates opacity to 0, y offscreen, scale to 0, or any other "out" animation before a transition fires. The transition IS the exit. The outgoing scene's content MUST be fully visible at the moment the transition starts.
4. **Final scene only:** The last scene may fade elements out (e.g., fade to black). This is the ONLY scene where `gsap.to(..., { opacity: 0 })` is allowed.

**WRONG — exit animation before transition:**

```js
// BANNED — this empties the scene before the transition can use it
tl.to("#s1-title", { opacity: 0, y: -40, duration: 0.4 }, 6.5);
tl.to("#s1-subtitle", { opacity: 0, duration: 0.3 }, 6.7);
// transition fires on empty frame
```

**RIGHT — entrance only, transition handles exit:**

```js
// Scene 1 entrance animations
tl.from("#s1-title", { y: 50, opacity: 0, duration: 0.7, ease: "power3.out" }, 0.3);
tl.from("#s1-subtitle", { y: 30, opacity: 0, duration: 0.5, ease: "power2.out" }, 0.6);
// NO exit tweens — transition at 7.2s handles the scene change
// Scene 2 entrance animations
tl.from("#s2-heading", { x: -40, opacity: 0, duration: 0.6, ease: "expo.out" }, 8.0);
```

## Animation Guardrails

- Offset first animation 0.1-0.3s (not t=0)
- Vary eases across entrance tweens — use at least 3 different eases per scene
- Don't repeat an entrance pattern within a scene
- Avoid full-screen linear gradients on dark backgrounds (H.264 banding — use radial or solid + localized glow)
- 60px+ headlines, 20px+ body, 16px+ data labels for rendered video
- `font-variant-numeric: tabular-nums` on number columns

If no `frame.md` or `design.md` exists, follow [house-style.md](./house-style.md) for aesthetic defaults.

## Typography and Assets

- **Built-in fonts:** Write the `font-family` you want in CSS — the compiler embeds supported fonts automatically.
- **Custom fonts:** If the spec (`frame.md` or `design.md`) names a font that isn't built-in, the user must provide `.woff2` files in a `fonts/` directory. If missing, warn before writing HTML. When files exist, add `@font-face` declarations pointing to the local files.
- Add `crossorigin="anonymous"` to external media
- For dynamic text overflow, use `window.__framevideo.fitTextFontSize(text, { maxWidth, fontFamily, fontWeight })`
- All files live at the project root alongside `index.html`; sub-compositions use `../`

## Editing Existing Compositions

- **Read actual files, don't guess.** When editing, extending, or creating companion compositions, read the existing source. Don't reconstruct hex codes from memory. Don't guess GSAP easing patterns. The composition IS the spec — extract exact values from it.
- Match existing fonts, colors, animation patterns from what you read
- Only change what was requested
- Preserve timing of unrelated clips

## Output Checklist

## Chanjing OAuth During Authoring

When a requested video needs Chanjing-backed features (public voices, platform speech, digital humans, or any Studio panel reporting missing Chanjing auth), invoke the `chanjing-digital-human` skill — especially **Auth Contract** and **Missing Credentials UX**. Do not stop at terminal env-var guidance when preview/Studio login is available.

Never print, echo, or commit tokens or credential-store contents. Do not fake generated assets when auth is missing; continue with placeholders only if the user accepts that as an interim state.

## Output Checklist

**Fast (run immediately, block on results):**

- [ ] `npx framevideo lint` and `npx framevideo validate` both pass
- [ ] Design adherence verified if a design spec (`frame.md` or `design.md`) exists

**Slow (run in parallel while presenting the preview to the user):**

- [ ] `npx framevideo inspect` passes, or every reported overflow is intentionally marked
- [ ] Contrast warnings addressed (see Quality Checks below)
- [ ] Animation choreography verified (see Quality Checks below)
- [ ] `framevideo-visual-qa` applied for new compositions and significant visual changes

## Quality Checks

### Visual Inspect

`framevideo inspect` runs the composition in headless Chrome, seeks through the timeline, and maps visual layout issues with timestamps, selectors, bounding boxes, and fix hints. Run it after `lint` and `validate`:

```bash
npx framevideo inspect
npx framevideo inspect --json
```

Failures usually mean text is spilling out of a bubble/card, a fixed-size label is clipping dynamic copy, or text has moved off the canvas. Fix by increasing container size or padding, reducing font size or letter spacing, adding a real `max-width` so text wraps inside the container, or using `window.__framevideo.fitTextFontSize(...)` for dynamic copy.

Use `--samples 15` for dense videos and `--at 1.5,4,7.25` for specific hero frames. Repeated static issues are collapsed by default to avoid flooding agent context. If overflow is intentional for an entrance/exit animation, mark the element or ancestor with `data-layout-allow-overflow`. If a decorative element should never be audited, mark it with `data-layout-ignore`.

`framevideo layout` is the compatibility alias for the same check.

### Contrast

`framevideo validate` runs a WCAG contrast audit by default. It seeks to 5 timestamps, screenshots the page, samples background pixels behind every text element, and computes contrast ratios. Failures appear as warnings:

```
⚠ WCAG AA contrast warnings (3):
  · .subtitle "secondary text" — 2.67:1 (need 4.5:1, t=5.3s)
```

If warnings appear:

- On dark backgrounds: brighten the failing color until it clears 4.5:1 (normal text) or 3:1 (large text, 24px+ or 19px+ bold)
- On light backgrounds: darken it
- Stay within the palette family — don't invent a new color, adjust the existing one
- Re-run `framevideo validate` until clean

Use `--no-contrast` to skip if iterating rapidly and you'll check later.

### Design Adherence

If a design spec (`frame.md` or `design.md`) exists, verify the composition follows it after authoring. Read the HTML and check:

1. **Colors** — every hex value in the composition appears in the spec's palette section (however the user labeled it: Colors, Palette, Theme, etc.). Flag any invented colors.
2. **Typography** — font families and weights match the spec's type spec. No substitutions.
3. **Corners** — border-radius values match the declared corner style, if specified.
4. **Spacing** — padding and gap values fall within the declared density range, if specified.
5. **Depth** — shadow usage matches the declared depth level, if specified (flat = none, subtle = light, layered = glows).
6. **Avoidance rules** — if the spec has a section listing things to avoid (commonly "What NOT to Do", "Don'ts", "Anti-patterns", or "Do's and Don'ts"), verify none are present.

Report violations as a checklist. Fix each one before serving.

If no design spec exists (house-style-only path), verify:

1. **Palette consistency** — the same bg, fg, and accent colors are used across all scenes. No per-scene color invention.
2. **No lazy defaults** — check the composition against house-style.md's "Lazy Defaults to Question" list. If any appear, they must be a deliberate choice for the content, not a default.

### Animation Map

After authoring animations, run the animation map to verify choreography:

```bash
node skills/framevideo/scripts/animation-map.mjs <composition-dir> \
  --out <composition-dir>/.framevideo/anim-map
```

Outputs a single `animation-map.json` with:

- **Per-tween summaries**: `"#card1 animates opacity+y over 0.50s. moves 23px up. fades in. ends at (120, 200)"`
- **ASCII timeline**: Gantt chart of all tweens across the composition duration
- **Stagger detection**: reports actual intervals (`"3 elements stagger at 120ms"`)
- **Dead zones**: periods over 1s with no animation — intentional hold or missing entrance?
- **Element lifecycles**: first/last animation time, final visibility
- **Scene snapshots**: visible element state at 5 key timestamps
- **Flags**: `offscreen`, `collision`, `invisible`, `paced-fast` (under 0.2s), `paced-slow` (over 2s)

Read the JSON. Scan summaries for anything unexpected. Check every flag — fix or justify. Verify the timeline shows the intended choreography rhythm. Re-run after fixes.

Skip on small edits (fixing a color, adjusting one duration). Run on new compositions and significant animation changes.

---

## References (loaded on demand)

- **`framevideo-visual-qa` skill** — Visual QA for safe areas, overlap, overflow, contrast, dynamic text fitting, and motion collisions. Use after new compositions or major layout/caption/asset changes.
- **[references/captions.md](references/captions.md)** — Captions, subtitles, lyrics, karaoke synced to audio. Tone-adaptive style detection, per-word styling, text overflow prevention, caption exit guarantees, word grouping. Read when adding any text synced to audio timing.
- **[references/audio-reactive.md](references/audio-reactive.md)** — Audio-reactive animation: map frequency bands and amplitude to GSAP properties. Read when visuals should respond to music, voice, or sound.
- **[references/css-patterns.md](references/css-patterns.md)** — CSS+GSAP marker highlighting: highlight, circle, burst, scribble, sketchout. Deterministic, fully seekable. Read when adding visual emphasis to text.
- **[references/video-composition.md](references/video-composition.md)** — Video-medium rules: density, color presence, scale, frame composition, the design spec as brand not layout. **Always read** — these override web instincts.
- **[references/beat-direction.md](references/beat-direction.md)** — Beat planning: concept, mood, choreography verbs, rhythm templates, transition decisions, depth layers. **Always read for multi-scene compositions.**
- **[references/typography.md](references/typography.md)** — Typography: font pairing, OpenType features, dark-background adjustments, font discovery script. **Always read** — every composition has text.
- **[references/motion-principles.md](references/motion-principles.md)** — Motion design principles, image motion treatment, load-bearing GSAP rules. **Always read** — every composition has motion.
- **[references/techniques.md](references/techniques.md)** — 13 primitive animation techniques with code patterns: SVG drawing, Canvas 2D, CSS 3D, kinetic type, Lottie, video compositing, typing, variable fonts, MotionPath, velocity transitions, audio-reactive, clip-path reveals, WebGL shaders. Adapt the patterns — don't copy-paste. (For pre-built UI templates — terminal chrome, device mockups, moodboard layouts — see `registry/blocks/`.)
- **[references/html-in-canvas-patterns.md](references/html-in-canvas-patterns.md)** — HTML-in-Canvas patterns: live DOM as GPU texture via `drawElementImage` + `layoutsubtree`. Shared boilerplate + ~6 effect recipes (iPhone/MacBook mockups, liquid glass, magnetic, portal, shatter, text cursor). Use for 1–3 hero beats per video.
- **[references/narration.md](references/narration.md)** — Pacing, tone, script structure, number pronunciation, opening line patterns. Read when the composition includes voiceover or TTS.
- **[references/design-picker.md](references/design-picker.md)** — Create a design.md via visual picker. Read when no `frame.md` or `design.md` exists and the user wants to create one.
- **[visual-styles.md](visual-styles.md)** — 8 named visual styles with hex palettes, GSAP easing signatures, and shader pairings. Read when user names a style or when generating a design spec.
- **[house-style.md](house-style.md)** — Default motion, sizing, and color palettes when no `frame.md` or `design.md` is specified.
- **[patterns.md](patterns.md)** — PiP, title cards, slide show patterns.
- **[data-in-motion.md](data-in-motion.md)** — Data, stats, and infographic patterns.
- **[references/transcript-guide.md](references/transcript-guide.md)** — Caption-side transcript handling: input formats, mandatory quality check, cleaning JS, OpenAI/Groq API fallback, "if no transcript exists" flow. (For the `transcribe` CLI invocation, model selection rules, and the `.en` gotcha, see the `framevideo-media` skill.)
- **[references/dynamic-techniques.md](references/dynamic-techniques.md)** — Dynamic caption animation techniques (karaoke, clip-path, slam, scatter, elastic, 3D).

- **[references/transitions.md](references/transitions.md)** — Scene transitions: crossfades, wipes, reveals, shader transitions. Energy/mood selection, CSS vs WebGL guidance. **Always read for multi-scene compositions** — scenes without transitions feel like jump cuts.
  - [transitions/catalog.md](references/transitions/catalog.md) — Hard rules, scene template, and routing to per-type implementation code.
  - Shader transitions are in `@framevideo/shader-transitions` (`packages/shader-transitions/`) — read package source, not skill files.

GSAP patterns and effects are in the `/gsap` skill.
