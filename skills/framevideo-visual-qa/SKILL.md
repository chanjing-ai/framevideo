---
name: framevideo-visual-qa
description: Visual quality review for FrameVideo compositions. Use when checking or fixing material overlap, text overlap, clipped or overflowing text, low text/background contrast, unsafe title/logo/CTA/caption placement, dynamic text fitting, visual collisions during animation, or final composition QA after creating or significantly editing FrameVideo HTML.
---

# FrameVideo Visual QA

Use this skill as the visual quality gate for FrameVideo work. It does not define brand identity. `frame.md`, `design.md`, or `DESIGN.md` define the project's colors, fonts, logo rules, and brand-specific constraints. This skill defines the reusable video QA method: layout safety, legibility, contrast, safe areas, and motion collisions.

## When To Run

Run this after creating a new composition, after changing layout/captions/assets/layering, before saying a video is ready, or whenever the user reports overlap, clipping, unreadable text, bad contrast, or unsafe placement.

For tiny edits, run only the checks related to the edit. For new or substantially changed compositions, run the full gate below.

## Gate

1. **Hero frame first** — identify the most crowded readable moment for each scene: all key text, foreground assets, captions, CTA, and logo visible. Inspect that frame before trusting the animation.
2. **Timeline samples next** — inspect entrance, hold, and near-exit frames for each scene. Dynamic problems often happen between the static start and final hold.
3. **Tools last** — run the commands and treat their output as evidence:

```bash
npx framevideo lint
npx framevideo validate
npx framevideo inspect
```

For dense videos, use:

```bash
npx framevideo inspect --samples 15
npx framevideo inspect --at 1.5,4,7.25
```

Use `--at` for known hero frames, caption-heavy moments, CTA holds, or frames the user called out.

## Safe Areas

Keep key information inside the safe area by default:

- Critical text, logo, CTA, captions, presenter faces, product UI, and legal/price copy stay away from edges.
- For 16:9 compositions, reserve at least about 5% of canvas width/height on all sides for key content. On a 1920x1080 frame, this is roughly 96px horizontal and 54px vertical.
- Use a more conservative bottom area for captions and CTA, especially when a progress bar, platform UI, or lower third may exist.
- Full-bleed backgrounds, glows, grain, particles, wipes, and transition matter may extend beyond the frame. Mark intentional overflow with `data-layout-allow-overflow` when `inspect` would otherwise report it.
- Mark decorative elements that should not be audited with `data-layout-ignore`.

Do not put readable copy, logos, or important product UI in the outermost 5% unless the user explicitly asks for an edge-anchored design and the frame has been visually checked.

## Contrast

Use `npx framevideo validate` as the contrast gate. It runs a WCAG AA audit by default.

- Normal text must clear 4.5:1.
- Large text must clear 3:1. Treat large text as 24px or larger, or 19px or larger when bold.
- On dark backgrounds, brighten the existing foreground color.
- On light backgrounds, darken the existing foreground color.
- Stay inside the project's palette family. Do not invent a new brand color just to pass contrast.
- If a warning samples a transition frame or a pre-entrance invisible element, verify that exact timestamp before dismissing it.

For every contrast warning, decide one of:

- **REAL ISSUE** — text is visible and too low contrast; fix the color, backing plate, shadow, scrim, or placement.
- **SAMPLING ARTIFACT** — text is not yet readable, is mid-fade, or is intentionally hidden at that timestamp; document why.

## Layout And Text

Build and review the readable end-state before adding or judging animation.

- Prefer `flex`, `grid`, padding, gap, `max-width`, and explicit zones for content layout.
- Keep absolute positioning for decoratives, overlays, and deliberately pinned elements. Avoid absolute top/left layout for the main content stack.
- Text containers need real width constraints. Use `max-width`, natural wrapping, or `window.__framevideo.fitTextFontSize(...)` for dynamic copy.
- Do not rely on forced line breaks for variable text. Split long copy into multiple scenes instead.
- If text is 35+ words in one scene, split it.
- If a label or badge has dynamic text, make the container elastic or fit the font. Fixed-size badges are common clipping sources.
- Captions must not cover CTA, logo, presenter face/body, product UI, or legally important copy.

Run `inspect` whenever text, captions, user-provided copy, variables, or translated strings can change.

## Overlap And Layering

Visual overlap is acceptable only when it is intentional and readable: shadows, glows behind text, z-stacked cards, texture, depth, or transition effects.

Flag and fix these as defects:

- Two readable text blocks occupy the same area at the same time.
- Caption text collides with lower thirds, CTA, logo, presenter, or product UI.
- A foreground asset covers the subject of an image or video.
- B-roll hides the message it is supposed to support.
- A decorative layer reduces text contrast enough to fail or feel muddy.
- An element is partially off-canvas during its readable hold.

If two elements reuse the same visual area, element A must finish its readable exit before element B begins its readable entrance, unless the storyboard calls for a deliberate overlap.

## Motion Collision

For significant animation changes, run the animation map from the installed FrameVideo skill directory:

```bash
node .agents/skills/framevideo/scripts/animation-map.mjs <project-dir> --out <project-dir>/.framevideo/anim-map
```

If the project has skills installed elsewhere, locate `framevideo/scripts/animation-map.mjs` in that skills directory and run the same script.

Read `animation-map.json` and check every `collision`, `offscreen`, `invisible`, `paced-fast`, `paced-slow`, and long dead-zone flag. Fix or explicitly justify each one. Do not use a green `lint` result as proof that motion is visually safe.

## Website And Digital-Human Videos

For website-derived videos, captured brand assets must support the message without hiding it:

- Do not cover core headline text with hero screenshots, SVGs, or B-roll.
- Do not let product UI sit behind same-colored text unless a scrim or backing plate preserves contrast.
- Keep captions and CTA clear of digital-human faces, hands, body, and subtitle tracks.
- Muted B-roll is a visual layer, not a license to obscure the primary message.

## Done Criteria

A composition passes visual QA only when:

- `npx framevideo lint` has no errors.
- `npx framevideo validate` has no unaddressed runtime errors or real WCAG AA failures.
- `npx framevideo inspect` has no unaddressed overflow issues, or intentional issues are marked with `data-layout-allow-overflow` / `data-layout-ignore`.
- Hero frames and sampled timeline frames show readable text, safe placement, and no unintentional overlap.
- Complex animation changes have an animation-map review with every flag fixed or justified.
