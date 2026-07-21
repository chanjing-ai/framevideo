---
name: lottie
description: Lottie and dotLottie adapter patterns for FrameVideo. Use when embedding lottie-web JSON animations, .lottie files, @lottiefiles/dotlottie-web players, registering instances on window.__fvLottie, or making After Effects exports deterministic in FrameVideo.
---

# Lottie for FrameVideo

## When To Use

Use Lottie for:

- **After Effects exports** — designer provides .json or .lottie file
- **Logo animations** — animated brand marks and icons
- **Pre-made animations** — using LottieFiles library assets
- **Complex vector motion** — shape morphing, path animations
- **Design handoff** — designers work in AE, devs integrate directly

## Do NOT Use

Avoid Lottie for:

- **Code-driven animation** — use `gsap` (more flexible)
- **Simple transforms** — use `gsap` or `css-animations` (lighter)
- **3D scenes** — use `three`
- **Text-heavy animation** — Lottie handles text poorly, use HTML + GSAP
- **Dynamic content** — Lottie animations are static, use programmatic animation

---

## Quick Start

Basic lottie-web animation:

```html
<div id="logo-lottie" class="lottie-layer"></div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/bodymovin/5.12.2/lottie.min.js"></script>
<script>
  const anim = lottie.loadAnimation({
    container: document.getElementById("logo-lottie"),
    renderer: "svg",
    loop: false,        // REQUIRED for FrameVideo
    autoplay: false,    // REQUIRED for FrameVideo
    path: "assets/logo-reveal.json",
  });

  window.__fvLottie = window.__fvLottie || [];
  window.__fvLottie.push(anim);
</script>

<style>
  .lottie-layer { width: 100%; height: 100%; }
</style>
```

**Key points:**
1. Set `autoplay: false` and `loop: false`
2. Register on `window.__fvLottie` array
3. Load from local `assets/`, not remote URLs

---

## Contract

- Load assets from local project files, usually under `assets/`.
- Set `autoplay: false`.
- Prefer `loop: false` unless the user explicitly wants a loop.
- Register every returned animation or player on `window.__fvLottie`.
- Keep the Lottie container dimensions stable with CSS.

The adapter seeks `lottie-web` with `goToAndStop(timeMs, false)` and dotLottie with frame or percentage APIs depending on player shape.

## lottie-web Pattern

```html
<div id="logo-lottie" class="lottie-layer"></div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/bodymovin/5.12.2/lottie.min.js"></script>
<script>
  const anim = lottie.loadAnimation({
    container: document.getElementById("logo-lottie"),
    renderer: "svg",
    loop: false,
    autoplay: false,
    path: "assets/logo-reveal.json",
  });

  window.__fvLottie = window.__fvLottie || [];
  window.__fvLottie.push(anim);
</script>
```

```css
.lottie-layer {
  width: 100%;
  height: 100%;
}
```

## dotLottie Pattern

```html
<canvas id="product-lottie" class="lottie-canvas"></canvas>
<script src="https://unpkg.com/@lottiefiles/dotlottie-web"></script>
<script>
  const player = new DotLottie({
    canvas: document.getElementById("product-lottie"),
    src: "assets/product-flow.lottie",
    autoplay: false,
    loop: false,
  });

  window.__fvLottie = window.__fvLottie || [];
  window.__fvLottie.push(player);
</script>
```

```css
.lottie-canvas {
  width: 100%;
  height: 100%;
  display: block;
}
```

## Multiple Animations

Push each player into the same registry:

```js
window.__fvLottie = window.__fvLottie || [];
window.__fvLottie.push(backgroundAnim);
window.__fvLottie.push(iconAnim);
window.__fvLottie.push(confettiAnim);
```

FrameVideo seeks them all to the same composition time.

## Good Uses

- After Effects exports that are already known to render correctly in lottie-web.
- Logo reveals, icon loops, decorative accents, and product UI motion.
- Translating Remotion Lottie usage into plain FrameVideo HTML.

## Avoid

- Relying on remote `path` URLs at render time.
- Starting playback with `play()`.
- Assuming unsupported After Effects effects will survive export. Test the JSON or `.lottie` file in a browser first.
- Loading a player asynchronously and registering it after FrameVideo validation has already inspected the page.

## Validation

After editing a Lottie composition:

```bash
npx framevideo lint
npx framevideo validate
```

## Credits And References

- FrameVideo adapter source: `packages/core/src/runtime/adapters/lottie.ts`.
- lottie-web by Airbnb: https://github.com/airbnb/lottie-web
- lottie-web `loadAnimation` options: https://github.com/airbnb/lottie-web/wiki/loadAnimation-options
- dotLottie web player methods by LottieFiles: https://developers.lottiefiles.com/docs/dotlottie-player/dotlottie-web/methods
