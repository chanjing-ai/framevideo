---
name: website-to-framevideo
description: |
  Capture a website and create a FrameVideo video from it. Use when: (1) a user provides a URL and wants a video, (2) someone says "capture this site", "turn this into a video", "make a promo from my site", (3) the user wants a social ad, product tour, or any video based on an existing website, (4) the user shares a link and asks for any kind of video content. Even if the user just pastes a URL — this is the skill to use.
---

# Website to FrameVideo

Orchestrate the 7-step workflow to capture a website and produce a professional video. This skill is the **workflow coordinator** — each step loads the appropriate skill or reference for implementation details.

## When To Use

- User provides a URL and wants a video (product demo, social ad, brand reel)
- User says "capture this site", "turn this into a video", "make a promo"
- User wants a video based on existing website content

## Quick Start

Basic workflow for website → video:

```bash
# User provides URL
"Make a 30-second product demo video from https://example.com"

# System orchestrates:
1. Capture site (Step 0) → extract brand + content
2. Create DESIGN.md (Step 1) → lock visual identity
3. Align on message (Step 2) → get video brief approval 💬
4. Storyboard + script (Step 3) → get narrative approval 💬
5. Generate voiceover (Step 4) → produce audio + timing
6. Build HTML (Step 5) → create FrameVideo composition
7. Validate + deliver (Step 6) → QA + Studio URL
```

**Key points:**
- 💬 = requires user approval gate
- Each step loads specific skills/references progressively
- Autonomous mode still requires verification (no skipping QA)

---

## Workflow Overview

Each step produces an artifact that gates the next. Gates marked 💬 require user approval. The workflow loads skills and references progressively — only read what's needed for the current step.

```
Step 0: Capture & Brand       → Site summary
Step 1: Brand Identity         → DESIGN.md
Step 2: Strategy & Messaging   → Video brief 💬
Step 3: Storyboard + Script    → STORYBOARD.md + SCRIPT.md 💬
Step 4: VO, Timing + Captions  → Audio + transcript 💬
Step 5: Build Compositions     → index.html + compositions
Step 6: Validate & Deliver     → Studio URL
```

### Autonomous Mode Rules

If user signals autonomous mode ("decide for me", "surprise me"):
- ✅ **Auto-decide:** TTS provider, voice, colors, beat count, music, captions
- ❌ **Still required:** Asset audit, per-beat HTML review, DoD checklist, verification disclosure

Autonomous mode = decide preferences, NOT skip verification.

---

## Step 0: Capture & Understand the Brand

**Load:** [references/step-0-capture.md](references/step-0-capture.md)

Capture the website using appropriate tool, extract brand assets (colors, fonts, images, copy), understand what the product does and who it's for.

**Gate:** Print site summary — strategy-first (product value, audience, brand voice) before asset inventory.

---

## Step 1: Brand Identity

**Load:** [references/step-1-design.md](references/step-1-design.md)

Write DESIGN.md covering visual identity: color palette, typography, component styles, layout principles. Use `design-styles.json` for exact computed values.

**Gate:** `DESIGN.md` exists with color palette, font choices, and do's/don'ts.

---

## Step 2: Strategy & Messaging

**Load:** [references/step-2-brief.md](references/step-2-brief.md), scan [references/capabilities.md](references/capabilities.md) TOC

Align with user on **what the video must communicate**. Parse user's prompt for video type and style. Lock the message and narrative arc before storyboarding.

**Gate:** Video type, duration, format, message, and narrative arc are locked.

---

## Step 3: Storyboard + Script 💬

**Load:** [references/step-3-storyboard.md](references/step-3-storyboard.md)

Write storyboard concept-first: message → narrative arc → beats → techniques per beat → brand accents. Write narration script to match. Present both to user for approval.

**Gate:** `STORYBOARD.md` + `SCRIPT.md` exist AND user has approved the plan.

---

## Step 4: VO, Timing + Captions 💬

**Load:** [references/step-4-vo.md](references/step-4-vo.md)
**Use:** `framevideo-media` for TTS generation, `framevideo-voiceover-ssml` if pronunciation control needed

If no narration requested: ask about background music, skip to Step 5. Otherwise: choose TTS provider, generate audio, transcribe, map timestamps to beats.

**Gate:** Either (a) no narration, manual beat timings in storyboard, or (b) `narration.wav` + `transcript.json` exist and beat timings updated.

---

## Step 5: Build Compositions

**Load:** `framevideo` skill (entire skill — every rule matters)
**Load:** [references/step-5-build.md](references/step-5-build.md)
**Use:** `gsap` for animation (default), other animation adapters as needed

Build index.html and beat compositions following storyboard architecture. Sub-agents run `framevideo lint` and `framevideo snapshot` on each beat.

**Gate:** Every `compositions/beat-N.html` has been read top-to-bottom by main agent against DESIGN.md and STORYBOARD.md.

---

## Step 6: Validate & Deliver

**Load:** [references/step-6-validate.md](references/step-6-validate.md)
**Use:** `framevideo-visual-qa` for final review, `framevideo-cli` for validation

Run lint, validate, inspect. Take snapshots (formula: `max(beats × 3, ceil(duration_seconds / 2))`). Review each snapshot. Use visual QA to confirm brand assets don't obscure message and captions don't collide with content. Fix issues before delivering.

**Deliver something you're proud of.** Ask: would I post this with my name on it?

**Gate:** `npx framevideo lint`, `validate`, `inspect` pass with zero unaddressed issues, visual QA passes, final response includes Studio URL.

---

## Quick Reference

### Typical Video Formats
- **Landscape:** 1920x1080 (default)
- **Portrait:** 1080x1920 (Instagram Stories, TikTok)
- **Square:** 1080x1080 (Instagram feed)

### Reference Files

| File | When to Load |
|------|--------------|
| [step-0-capture.md](references/step-0-capture.md) | Step 0: Capture site, write brand summary |
| [step-1-design.md](references/step-1-design.md) | Step 1: Write DESIGN.md |
| [step-2-brief.md](references/step-2-brief.md) | Step 2: Align on message and arc |
| [capabilities.md](references/capabilities.md) | Steps 2 & 5: Scan TOC, deep-dive as needed |
| [step-3-storyboard.md](references/step-3-storyboard.md) | Step 3: Write storyboard + script |
| [step-4-vo.md](references/step-4-vo.md) | Step 4: Generate audio and timing |
| [step-5-build.md](references/step-5-build.md) | Step 5: Build compositions |
| [step-6-validate.md](references/step-6-validate.md) | Step 6: Validate and deliver |

### Skills Loaded by Step

| Step | Skills |
|------|--------|
| 0 | (capture tools) |
| 1 | (file operations) |
| 2 | (alignment, no skills) |
| 3 | (planning, no skills) |
| 4 | `framevideo-media`, `framevideo-voiceover-ssml` (optional) |
| 5 | `framevideo`, `gsap`, animation adapters as needed |
| 6 | `framevideo-cli`, `framevideo-visual-qa` |

---

## Integration

This skill orchestrates:
- **framevideo** — core composition authoring (Step 5)
- **framevideo-cli** — validation commands (Step 6)
- **framevideo-media** — TTS and transcription (Step 4)
- **framevideo-voiceover-ssml** — pronunciation control (Step 4, optional)
- **framevideo-visual-qa** — final quality review (Step 6)
- **gsap** — animation (Step 5, default adapter)
- **chanjing-digital-human** — if user wants digital human host (Step 4-5)

---

## Validation

After completing workflow:
- [ ] All gates passed
- [ ] `npx framevideo lint` passes
- [ ] `npx framevideo validate` passes
- [ ] `npx framevideo inspect` passes
- [ ] Visual QA checklist complete
- [ ] Studio URL delivered
- [ ] "What I did NOT verify" section included in summary

---

## Credits And References

- FrameVideo documentation: https://framevideo.dev
- Workflow references: `references/` directory in this skill
