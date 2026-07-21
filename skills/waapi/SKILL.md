---
name: waapi
description: Web Animations API adapter patterns for FrameVideo. Use when authoring element.animate() motion, Animation currentTime seeking, document.getAnimations(), KeyframeEffect timing, fill modes, or native browser animations that must render deterministically in FrameVideo.
---

# Web Animations API for FrameVideo

## When To Use

Use WAAPI for:

- **Zero-dependency animations** — no external libraries needed
- **Lightweight DOM motion** — simple transforms and opacity changes
- **Native browser performance** — browser-optimized keyframe engine
- **Generated animations** — creating animations from structured data
- **Minimal bundle size** — when every KB counts

## Do NOT Use

Avoid WAAPI for:

- **Complex timeline sequencing** — use `gsap` (better timeline control)
- **Default choice** — use `gsap` unless you specifically want zero dependencies
- **CSS-only decoration** — use `css-animations` (simpler)
- **3D/WebGL** — use `three`
- **Designer exports** — use `lottie`

---

## Quick Start

Basic WAAPI animation in FrameVideo:

```html
<div id="orb" class="clip orb" data-start="2" data-duration="3">Animate me</div>

<script>
  const orb = document.getElementById("orb");
  const animation = orb.animate(
    [
      { transform: "translateX(-160px) scale(0.8)", opacity: 0 },
      { transform: "translateX(0) scale(1)", opacity: 1, offset: 0.35 },
      { transform: "translateX(120px) scale(1.08)", opacity: 1 },
    ],
    {
      duration: 3000,
      delay: 2000,
      easing: "cubic-bezier(0.2, 0, 0, 1)",
      fill: "both",        // REQUIRED for FrameVideo
      iterations: 1,
    },
  );

  animation.pause();     // Pause immediately
</script>
```

**Key points:**
1. Use `fill: "both"` to hold seeked states
2. Call `animation.pause()` after creation
3. Use finite `iterations`

---

## Contract

- Create animations synchronously during composition initialization.
- Use `element.animate(...)` with finite `duration` and `iterations`.
- Use `fill: "both"` so seeked states persist.
- Pause animations after creation or let the adapter pause them on first seek.
- Avoid callbacks and promises for render-critical state.

The adapter calls `document.getAnimations()`, sets each animation's `currentTime` to FrameVideo time in milliseconds, then pauses it.

## Basic Pattern

```html
<div id="orb" class="clip orb" data-start="2" data-duration="3" data-track-index="2"></div>

<script>
  const orb = document.getElementById("orb");
  const animation = orb.animate(
    [
      { transform: "translate3d(-160px, 0, 0) scale(0.8)", opacity: 0 },
      { transform: "translate3d(0, 0, 0) scale(1)", opacity: 1, offset: 0.35 },
      { transform: "translate3d(120px, 0, 0) scale(1.08)", opacity: 1 },
    ],
    {
      duration: 3000,
      delay: 2000,
      easing: "cubic-bezier(0.2, 0, 0, 1)",
      fill: "both",
      iterations: 1,
    },
  );

  animation.pause();
</script>
```

## Stagger Pattern

```js
document.querySelectorAll(".token").forEach((token, index) => {
  const animation = token.animate(
    [
      { transform: "translateY(24px)", opacity: 0 },
      { transform: "translateY(0)", opacity: 1 },
    ],
    {
      duration: 620,
      delay: index * 80,
      easing: "cubic-bezier(0.2, 0, 0, 1)",
      fill: "both",
      iterations: 1,
    },
  );
  animation.pause();
});
```

## Good Uses

- Lightweight DOM motion where CSS keyframes are too rigid and GSAP is unnecessary.
- Generated animations from structured data.
- Simple timelines that can be represented as keyframes, delays, and offsets.

## Avoid

- Infinite `iterations`.
- Depending on `animation.finished` to mutate render-critical DOM.
- Running separate clocks with `requestAnimationFrame`, timers, or `performance.now()`.
- Animating layout properties when transforms and opacity can express the motion.
- Assuming clip-local start time is automatic. WAAPI adapter seeks document-level animation time; model clip offsets with `delay` or create the animation on an element whose visibility is controlled by FrameVideo timing.

## Validation

After editing a WAAPI composition:

```bash
npx framevideo lint      # Check structure
npx framevideo validate  # Check runtime errors
npx framevideo preview   # Scrub timeline to verify seekability
```

**Manual checks:**
1. **Fill mode** — all animations use `fill: "both"`
2. **Paused state** — animations paused after creation
3. **Finite iterations** — no infinite loops
4. **Seekability** — scrub preview, animation holds at any frame
5. **Performance** — prefer transform/opacity over layout properties

## Credits And References

- FrameVideo adapter source: `packages/core/src/runtime/adapters/waapi.ts`.
- MDN Web Animations API guide: https://developer.mozilla.org/docs/Web/API/Web_Animations_API/Using_the_Web_Animations_API
- MDN `Animation.currentTime`: https://developer.mozilla.org/en-US/docs/Web/API/Animation/currentTime
