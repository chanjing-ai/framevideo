---
name: three
description: Three.js and WebGL adapter patterns for FrameVideo. Use when creating deterministic Three.js scenes, WebGL canvas layers, AnimationMixer timelines, camera motion, shader-driven visuals, or canvas renders that respond to FrameVideo fv-seek events.
---

# Three.js for FrameVideo

## When To Use

Use Three.js for:

- **3D product showcases** — rotating products, exploded views
- **3D scenes and environments** — backgrounds, animated 3D elements
- **GLTF model animation** — loading and animating 3D models
- **Camera motion** — dolly, pan, orbit camera movements
- **WebGL shader effects** — custom shaders, post-processing

## Do NOT Use

Avoid Three.js for:

- **2D animations** — use `gsap` (simpler and lighter)
- **GPU compute shaders** — use `typegpu` (better compute pipeline support)
- **Simple effects** — Three.js is heavy, use lighter alternatives when possible
- **Text animation** — use HTML + GSAP (better text rendering)

---

## Quick Start

Basic Three.js scene in FrameVideo:

```html
<canvas id="three-layer" style="position: absolute; inset: 0;"></canvas>

<script type="module">
  import * as THREE from "https://cdn.jsdelivr.net/npm/three@0.181.2/+esm";

  const canvas = document.getElementById("three-layer");
  const renderer = new THREE.WebGLRenderer({ canvas, alpha: true });
  renderer.setSize(1920, 1080, false);

  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(35, 1920/1080, 0.1, 100);
  camera.position.z = 6;

  const mesh = new THREE.Mesh(
    new THREE.BoxGeometry(2, 2, 2),
    new THREE.MeshStandardMaterial({ color: 0x64d2ff })
  );
  scene.add(mesh);
  scene.add(new THREE.HemisphereLight(0xffffff, 0x223344, 2));

  // Listen for FrameVideo seek events
  window.addEventListener("fv-seek", (event) => {
    const time = event.detail.time;
    mesh.rotation.y = time * 0.7;
    renderer.render(scene, camera);
  });
</script>
```

**Key points:**
1. Listen for `fv-seek` event
2. Render at `event.detail.time`
3. Avoid `requestAnimationFrame`

---

## Contract

- Create the scene, camera, renderer, materials, and assets synchronously when possible.
- Render from FrameVideo time, not wall-clock time.
- Listen for the `fv-seek` event and render exactly that time.
- Load models, textures, and HDRIs before render-critical seeking. Do not fetch them at seek time.
- Avoid `requestAnimationFrame` or `renderer.setAnimationLoop` as the source of truth for render-critical motion.

The adapter sets `window.__fvThreeTime` and dispatches `new CustomEvent("fv-seek", { detail: { time } })` on each seek.

## Basic Pattern

```html
<canvas id="three-layer"></canvas>
<script type="module">
  import * as THREE from "https://cdn.jsdelivr.net/npm/three@0.181.2/+esm";

  const canvas = document.getElementById("three-layer");
  const renderer = new THREE.WebGLRenderer({ canvas, alpha: true, antialias: true });
  // Match these to your composition's frame size.
  renderer.setSize(1920, 1080, false);
  renderer.setPixelRatio(1);

  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(35, 1920 / 1080, 0.1, 100);
  camera.position.set(0, 0, 6);

  const mesh = new THREE.Mesh(
    new THREE.IcosahedronGeometry(1.4, 4),
    new THREE.MeshStandardMaterial({ color: 0x64d2ff, roughness: 0.38 }),
  );
  scene.add(mesh);
  scene.add(new THREE.HemisphereLight(0xffffff, 0x223344, 2));

  function renderAt(time) {
    mesh.rotation.y = time * 0.7;
    mesh.rotation.x = Math.sin(time * 0.6) * 0.16;
    renderer.render(scene, camera);
  }

  window.addEventListener("fv-seek", (event) => {
    renderAt(event.detail.time);
  });

  renderAt(window.__fvThreeTime || 0);
</script>
```

```css
#three-layer {
  width: 100%;
  height: 100%;
  display: block;
}
```

## AnimationMixer Pattern

For GLTF or authored clip animation, seek the mixer directly:

```js
function renderAt(time) {
  mixer.setTime(time);
  renderer.render(scene, camera);
}
```

If several mixers exist, seek all of them from the same `time`.

## Good Uses

- Deterministic 3D objects, product spins, particles with seeded data, and shader plates.
- Camera moves derived from `time`.
- GLTF animation clips when assets are local and loaded before validation completes.

## Avoid

- Using `Date.now()`, `performance.now()`, or clock deltas to update scene state.
- Leaving render-critical work inside a free-running animation loop.
- Loading remote models or textures at render time.
- Device-pixel-ratio dependent output. Pin renderer size and pixel ratio for video renders.
- Post-processing passes that depend on previous frame history unless you can reconstruct state from time.

## Validation

After editing a Three.js composition:

```bash
npx framevideo lint
npx framevideo validate
```

## Credits And References

- FrameVideo adapter source: `packages/core/src/runtime/adapters/three.ts`.
- Three.js `WebGLRenderer` docs: https://threejs.org/docs/pages/WebGLRenderer.html
- Three.js `AnimationMixer.setTime()` docs: https://threejs.org/docs/pages/AnimationMixer.html
