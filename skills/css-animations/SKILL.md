---
name: css-animations
description: CSS animation adapter patterns for FrameVideo. Use when authoring CSS keyframes, animation-delay based timing, animation-fill-mode, animation-play-state, or CSS-only motion that FrameVideo must seek deterministically during preview and rendering.
---

# CSS Animations for FrameVideo

FrameVideo can seek CSS keyframe animations through its `css` runtime adapter. Use this for simple repeated motifs, background motion, shimmer, glow, masks, and non-sequenced decoration.

For scene choreography, GSAP is usually clearer. CSS animations work best when the motion belongs to one element and has a fixed duration.

## When To Use

Use CSS animations for:

- **Decorative layers** — background patterns, shimmer effects, grain, subtle parallax that doesn't need precise timing
- **Simple loops** — pulse, glow, breathing effects with known repeat counts
- **Per-element motion** — entrance animations where each element animates independently
- **Zero-dependency motion** — when you want animation without loading GSAP or other libraries
- **GPU-accelerated effects** — transform and opacity animations that browsers optimize automatically

## Do NOT Use

Avoid CSS animations for:

- **Scene choreography** — multi-element sequences with precise timing → use `gsap`
- **Complex timelines** — anything needing pause/resume control or dynamic duration → use `gsap`
- **Infinite loops** — unless verified that browser exposes seekable WAAPI handles (rare in headless render)
- **Interactive animations** — hover, scroll, or user-triggered motion won't render deterministically
- **Layout animations** — animating `top`, `left`, `width`, `height` causes reflow → use transforms instead

## Contract

- Put the animated element in the DOM before runtime initialization finishes.
- Give timed elements a `data-start` value so local animation time matches the clip.
- Use finite `animation-duration` and `animation-iteration-count` because the negative-delay fallback cannot represent unbounded duration in environments without WAAPI-backed CSS animations.
- Prefer `animation-fill-mode: both` so seeked states hold before and after active motion.
- Avoid wall-clock JavaScript, hover-triggered state, and class toggles that depend on user events.

The adapter discovers elements with computed `animation-name`, seeks their browser `Animation` handles when available, and falls back to pausing with negative `animation-delay`.

## Basic Pattern

```html
<div
  id="pulse-ring"
  class="clip pulse-ring"
  data-start="0"
  data-duration="4"
  data-track-index="2"
></div>

<style>
  .pulse-ring {
    width: 280px;
    height: 280px;
    border: 4px solid rgba(255, 255, 255, 0.7);
    border-radius: 50%;
    animation-name: pulse-ring;
    animation-duration: 1200ms;
    animation-timing-function: cubic-bezier(0.2, 0, 0, 1);
    animation-iteration-count: 3;
    animation-fill-mode: both;
  }

  @keyframes pulse-ring {
    from {
      opacity: 0;
      transform: scale(0.82);
    }
    35% {
      opacity: 1;
    }
    to {
      opacity: 0;
      transform: scale(1.18);
    }
  }
</style>
```

## Stagger Pattern

Use CSS custom properties to avoid duplicating keyframes:

```html
<div class="clip dots" data-start="1" data-duration="3" data-track-index="3">
  <span style="--i: 0"></span>
  <span style="--i: 1"></span>
  <span style="--i: 2"></span>
</div>

<style>
  .dots span {
    display: inline-block;
    width: 18px;
    height: 18px;
    margin-right: 10px;
    border-radius: 50%;
    background: currentColor;
    animation: dot-pop 900ms ease-out both;
    animation-delay: calc(var(--i) * 120ms);
  }

  @keyframes dot-pop {
    from {
      opacity: 0;
      transform: translateY(18px) scale(0.75);
    }
    to {
      opacity: 1;
      transform: translateY(0) scale(1);
    }
  }
</style>
```

## Good Uses

- Decorative loops with a known repeat count.
- Mask, glow, shimmer, grain, and subtle parallax layers.
- Simple one-element entrances where a full JS timeline would be excessive.

## Combining with GSAP

CSS animations and GSAP timelines can coexist in the same composition. The division of labor:

**CSS handles decoration:**
- Background patterns, grain, noise
- Ambient glow, shimmer, pulse effects
- Mask reveals, vignettes
- Subtle environment motion (floating particles, slow drift)

**GSAP handles choreography:**
- Caption timing and sequencing
- Scene transitions and crossfades
- UI element entrances coordinated with audio
- Multi-element stagger with precise control

### Sync Pattern

CSS animation timing aligns with clip `data-start`:

```html
<div class="clip bg-shimmer" data-start="0" data-duration="10">
  <!-- CSS animation runs 0-10s, synced to clip -->
</div>

<script>
  const tl = gsap.timeline();
  tl.to("#caption-1", { opacity: 1, duration: 0.5 }, 2);
  tl.to("#caption-2", { opacity: 1, duration: 0.5 }, 5);
  // GSAP controls captions, CSS handles background
</script>
```

The CSS adapter and GSAP adapter run independently but both respect FrameVideo's global timeline, so they stay in sync during preview and render.

## Advanced Patterns

### Complex Stagger with Custom Properties

For stagger based on data attributes or computed values:

```html
<div class="grid">
  <div class="cell" style="--row: 0; --col: 0"></div>
  <div class="cell" style="--row: 0; --col: 1"></div>
  <!-- ... -->
</div>

<style>
  .cell {
    animation: cell-reveal 600ms ease-out both;
    animation-delay: calc((var(--row) * 3 + var(--col)) * 80ms);
  }

  @keyframes cell-reveal {
    from { opacity: 0; transform: scale(0.8); }
    to { opacity: 1; transform: scale(1); }
  }
</style>
```

### Finite Loops for Infinite Patterns

When a pattern looks infinite but must be finite for rendering:

```css
.background-drift {
  /* Appears infinite but is actually 50 iterations over 100s */
  animation: drift 2000ms linear 50;
}

@keyframes drift {
  from { transform: translateX(0); }
  to { transform: translateX(-100px); }
}
```

**Rule:** Calculate iteration count = `clip duration / animation duration`, rounded up. For a 30s clip with 2s animation: `animation-iteration-count: 15`.

### Animation Chains

Run multiple animations sequentially on one element:

```css
.logo {
  animation: 
    fade-in 800ms ease-out 0s both,
    pulse 1200ms ease-in-out 1s 3 both,
    fade-out 800ms ease-in 4.6s both;
}

@keyframes fade-in { from { opacity: 0; } to { opacity: 1; } }
@keyframes pulse { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.05); } }
@keyframes fade-out { from { opacity: 1; } to { opacity: 0; } }
```

Delays stack: fade-in at 0s, pulse at 1s, fade-out at 4.6s.

### Performance Optimization

```css
.optimized {
  /* Tell browser to promote to GPU layer */
  will-change: transform, opacity;
  
  /* Prefer transform over position */
  animation: slide-in 600ms ease-out both;
}

@keyframes slide-in {
  /* Good: GPU-accelerated */
  from { transform: translateX(-100px); opacity: 0; }
  to { transform: translateX(0); opacity: 1; }
  
  /* Avoid: causes reflow */
  /* from { left: -100px; } to { left: 0; } */
}
```

Only animate `transform` and `opacity` when possible. Avoid `width`, `height`, `top`, `left`, `margin`, `padding`.

## Performance Comparison

| Metric | CSS Animations | GSAP | WAAPI | Anime.js |
|--------|----------------|------|-------|----------|
| Bundle size | 0 KB | 45 KB | 0 KB | 17 KB |
| Init time | 0 ms | 8 ms | 0 ms | 4 ms |
| Seek performance | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| GPU acceleration | Auto | Manual | Auto | Manual |
| Best for | Decoration | Sequencing | Lightweight | Compact |

**When CSS is faster:**
- Simple loops (pulse, glow, rotate)
- GPU-accelerated properties only (transform, opacity)
- No timeline control needed

**When GSAP is faster:**
- Complex sequences with 10+ tweens
- Precise timing control and labels
- Dynamic duration or playback speed
- Nested timelines

## Avoid

- Infinite CSS animations unless you have verified the browser exposes seekable WAAPI-backed CSS animation handles. Prefer a finite iteration count covering the visible duration.
- Animating layout properties like `top`, `left`, `width`, or `height` when transforms work.
- Relying on hover, focus, scroll, or media queries to trigger render-critical motion.
- Changing animation classes after startup unless another deterministic timeline controls that change.

## Validation

After editing CSS animation compositions:

```bash
npx framevideo lint      # Check timing contract
npx framevideo validate  # Check runtime errors
```

**Manual checks:**

1. **Seek test** — scrub preview timeline, verify animation holds at each frame
2. **Iteration count** — ensure `animation-iteration-count` covers clip duration (no blank frames at end)
3. **Fill mode** — confirm `animation-fill-mode: both` so seeked states hold before/after
4. **GPU layers** — open DevTools Performance, record render, verify no layout thrashing

## Real-World Examples

### Example 1: Background Shimmer

**Use case:** Add subtle shimmer to product card background without loading animation library.

```html
<div class="clip product-card" data-start="0" data-duration="5">
  <div class="shimmer-layer"></div>
  <h2>Product Name</h2>
</div>

<style>
  .shimmer-layer {
    position: absolute;
    inset: 0;
    background: linear-gradient(90deg, transparent, rgba(255,255,255,0.2), transparent);
    animation: shimmer 2s linear infinite;
  }
  @keyframes shimmer {
    from { transform: translateX(-100%); }
    to { transform: translateX(100%); }
  }
</style>
```

**Result:** Smooth shimmer effect, 0 KB overhead, GPU-accelerated.

### Example 2: Loading Dots

**Use case:** Animated loading indicator during composition build phase.

```html
<div class="dots">
  <span style="--i: 0"></span>
  <span style="--i: 1"></span>
  <span style="--i: 2"></span>
</div>

<style>
  .dots span {
    animation: dot-bounce 1.4s ease-in-out infinite both;
    animation-delay: calc(var(--i) * 160ms);
  }
  @keyframes dot-bounce {
    0%, 80%, 100% { transform: scale(0); }
    40% { transform: scale(1); }
  }
</style>
```

**Result:** Classic three-dot animation with CSS-only stagger.

### Example 3: Pulse Ring on Logo

**Use case:** Draw attention to logo with pulsing ring effect.

```html
<div class="clip logo-intro" data-start="0" data-duration="4">
  <div class="logo-container">
    <img src="logo.svg" alt="Logo">
    <div class="pulse-ring"></div>
  </div>
</div>

<style>
  .pulse-ring {
    position: absolute;
    inset: -20px;
    border: 3px solid rgba(255,255,255,0.6);
    border-radius: 50%;
    animation: pulse 1.5s cubic-bezier(0.2, 0, 0, 1) 2 both;
  }
  @keyframes pulse {
    from { opacity: 0; transform: scale(0.8); }
    35% { opacity: 1; }
    to { opacity: 0; transform: scale(1.2); }
  }
</style>
```

**Result:** Logo appears with two expanding rings, total 3s animation.

## Credits And References

- FrameVideo adapter source: `packages/core/src/runtime/adapters/css.ts`.
- MDN CSS animation documentation: https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/animation
- MDN `animation-fill-mode`: https://developer.mozilla.org/en-US/docs/Web/CSS/animation-fill-mode
