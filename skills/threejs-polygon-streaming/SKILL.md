---
name: threejs-polygon-streaming
description: Polygon Streaming JavaScript SDK — Three.js binding. Integrating Viverse Polygon Streaming (.xrg) into a Three.js scene via the @polygon-streaming/web-player-threejs NPM package.
prerequisites: [Three.js scene with camera + renderer + animation loop, basis transcoder at /lib, service worker at web root]
tags: [polygon-streaming, viverse, threejs, xrg, streaming, vrm, javascript-sdk]
---

# Polygon Streaming JavaScript SDK — Three.js

Use this skill when integrating Viverse Polygon Streaming into a Three.js scene via `@polygon-streaming/web-player-threejs`. This is the **Polygon Streaming JavaScript SDK → Three.js** path in the official docs. The binding is an ES-module package driven by a per-frame `streamController.update()` call.

## When To Use This Skill

- Loading `.xrg` streamed models into a Three.js scene.
- Playing embedded animations, VRM animations (`.vrma`), or VRM expressions on a streamed model.
- Making the model's eyes track a target (VRM look-at).

## Preflight

- [ ] A Three.js scene with a `PerspectiveCamera`, a `WebGLRenderer`, and either `setAnimationLoop` or a `requestAnimationFrame` loop.
- [ ] KTX2 transcoder files copied to `/lib/basis_transcoder.js` and `/lib/basis_transcoder.wasm`.
- [ ] Service worker served at `/service-worker.js` (see below).
- [ ] `three` is installed and the version matches whatever your app already uses (Polygon Streaming does not pin Three).

## Package

```json
"dependencies": {
  "@polygon-streaming/web-player-threejs": "latest",
  "three": "^0.16x"
}
```

## Minimum Working Setup

```javascript
import * as THREE from 'three';
import { StreamController } from '@polygon-streaming/web-player-threejs';

// ...create scene, camera, renderer, and (optionally) OrbitControls as usual...

// 1. Stream controller — one per scene.
//    Fourth arg is the camera target (Vector3). For OrbitControls it's controls.target.
const streamController = new StreamController(
  camera,
  renderer,
  scene,
  controls.target,
  {
    cameraType: 'nonPlayer',
    triangleBudget: 5_000_000,
    mobileTriangleBudget: 3_000_000
  }
);

// 2. A Group that will host the streamed geometry.
const modelParent = new THREE.Group();
modelParent.position.set(0, 1, 0);
scene.add(modelParent);

// 3. Register the streamable model.
streamController.addModel(
  'https://stream.viverse.com/demos/jet-engine-11m/',
  modelParent,
  { qualityPriority: 1 }
);

// 4. Drive the stream controller from your animation loop.
function animate() {
  controls.update();
  renderer.render(scene, camera);
  streamController.update();     // REQUIRED every frame
}
renderer.setAnimationLoop(animate);
```

> [!CAUTION]
> `streamController.update()` **must** be called every frame. Without it, the model never streams in additional geometry beyond the initial load.

## Files Required at the Web Root

Polygon Streaming needs:

- `/service-worker.js` — chunk caching for `.xrg` fetches.
- `/lib/basis_transcoder.js` and `/lib/basis_transcoder.wasm` — KTX2 texture transcoder.

The `packages/threejs/example/vite.config.js` in the repo shows how to copy these using `vite-plugin-static-copy`. In another bundler, copy them from `@polygon-streaming/web-player-core` and `three/examples/jsm/libs/basis/` (or the KTX2 loader distribution) into your public directory during the build.

> [!CAUTION]
> Without the basis transcoder files at `/lib/`, textures fail to decode and meshes render either untextured or invisible. This is the most common Three.js integration bug.

## Playing an Embedded or VRM Animation

Once the initial model data has loaded (see `EVENT_MODEL_LOAD` below), you can play animations by targeting the model's parent `Group`:

```javascript
// arg 2: animation name (string) OR index (number, 0 = first)
// arg 3: transition duration in seconds (optional, default 1)
streamController.playAnimation(modelParent, 'idle', 0.2);
```

For VRM animations (`.vrma`), declare them up-front when calling `addModel`. Unlike the PlayCanvas binding, the Three.js binding takes VRMA files as **URLs**, not container assets:

```javascript
streamController.addModel('/model.xrg', modelParent, {
  vrmAnimations: [
    { name: 'idle', asset: '/Idle.vrma', default: true },
    { name: 'yawn', asset: '/Yawn.vrma', loop: false }
  ]
});
```

Loop / default semantics:

- `default: true` — the animation played automatically after any non-looping animation finishes. Almost always your idle.
- `loop: false` — one-shot (e.g. attack, wave). Returns to the default when done.
- `default` animations always loop, regardless of `loop`.

## Animating a VRM Expression

```javascript
streamController.animateVrmExpression(modelParent, 'happy', {
  times:  [0, 0.1, 0.2],   // seconds
  values: [0, 1,   0]       // expression weight 0..1
});
```

The expression name must exist in the source VRM that the `.xrg` was generated from.

## VRM Look-At (Eye Tracking)

`setVrmLookAtTarget` must be called **after** the initial model data has loaded — use the `EVENT_MODEL_LOAD` event. Only VRM-derived models react; the event's `isVrm` flag tells you.

```javascript
import { StreamController, EVENT_MODEL_LOAD }
  from '@polygon-streaming/web-player-threejs';

const vrmLookAtTarget = new THREE.Object3D();
camera.add(vrmLookAtTarget);

window.addEventListener('mousemove', (event) => {
  vrmLookAtTarget.position.x =
    10.0 * ((event.clientX - 0.5 * window.innerWidth) / window.innerHeight);
  vrmLookAtTarget.position.y =
    -10.0 * ((event.clientY - 0.5 * window.innerHeight) / window.innerHeight);
});

streamController.addEventListener(EVENT_MODEL_LOAD, (event) => {
  if (event.isVrm) {
    streamController.setVrmLookAtTarget(modelParent, vrmLookAtTarget);
  }
});
```

> [!CAUTION]
> Calling `setVrmLookAtTarget` before `EVENT_MODEL_LOAD` fires is a no-op — the VRM look-at rig is not yet constructed. Always gate it on the event.

## Stream Controller Parameters

Constructor: `new StreamController(camera, renderer, scene, cameraTarget, options)`

| Required arg | Type | Notes |
| --- | --- | --- |
| `camera` | `THREE.Camera` | The scene camera. |
| `renderer` | `THREE.WebGLRenderer` | The scene renderer. |
| `scene` | `THREE.Scene` | The root scene. |
| `cameraTarget` | `THREE.Vector3` | Where the camera is looking. `controls.target` if using `OrbitControls`. |

`options` (all optional):

| Option | Type | Default | Notes |
| --- | --- | --- | --- |
| `cameraType` | `'nonPlayer' \| 'player'` | `'nonPlayer'` | |
| `triangleBudget` | `number` | `5_000_000` | Global triangle cap. |
| `mobileTriangleBudget` | `number` | `3_000_000` | `0` = use desktop budget. |
| `minimumDistance` | `number` | `0.01` | |
| `distanceFactor` | `number` | `1.1` | >1 favors nearby geometry. |
| `maximumQuality` | `number` | `15000` | `0` = unlimited. |
| `closeUpDistance` | `number` | `3` | `0` disables close-up factor. |
| `closeUpDistanceFactor` | `number` | `5` | Should be > `distanceFactor`. |
| `iOSMemoryLimit` | `number` (MB) | `0` | `0` = auto, `-1` = no limit. |
| `showLoadingModel` | `boolean` | `true` | |
| `loadingModelUrl` | `string` | `null` | Override loading GLB. |

## Streamable Model Parameters

Method: `streamController.addModel(url, modelParent, options)`

| Required arg | Type | Notes |
| --- | --- | --- |
| `url` | `string` | URL of the `.xrg`. If it doesn't end in `.xrg`, `model.xrg` is appended. |
| `modelParent` | `THREE.Object3D` | A `Group` (or similar) already added to the scene. Streamed meshes are attached under this. |

`options`:

| Option | Type | Default | Notes |
| --- | --- | --- | --- |
| `qualityPriority` | `number` | `1` | Relative to other streamed models. |
| `initialTrianglePercent` | `number` | `0.1` | Portion of budget on first load. |
| `castShadows` | `boolean` | `true` | |
| `receiveShadows` | `boolean` | `true` | |
| `forceDoubleSided` | `boolean` | `false` | |
| `useAlpha` | `boolean` | `true` | Disable for opaque-only rendering. |
| `playAnimationAutomatically` | `boolean` | `true` | |
| `animation` | `string \| number` | `null` | Embedded animation name or index. |
| `vrmAnimations` | `Array<{ name, asset, loop?, default? }>` | `null` | `asset` is a **URL string** here. |
| `environmentMap` | `THREE.Texture` (cube) | `null` | |
| `hashCode` | `string` | `''` | |

## Verification Checklist

- [ ] `streamController.update()` runs every frame.
- [ ] `modelParent` was added to the scene **before** calling `addModel`.
- [ ] `/service-worker.js` is served at the web root in production.
- [ ] Basis transcoder files exist at `/lib/basis_transcoder.js` and `/lib/basis_transcoder.wasm`.
- [ ] `cameraTarget` is a live `Vector3` — if using `OrbitControls`, pass `controls.target` (not a copy).
- [ ] For VRM look-at: `setVrmLookAtTarget` is called inside an `EVENT_MODEL_LOAD` handler, only when `event.isVrm`.

## Gotchas

- **Missing `update()` call.** Model loads once and then never streams. Symptom: mesh looks low-poly and never improves as you move the camera.
- **Basis transcoder path.** Files must be at `/lib/basis_transcoder.js` and `/lib/basis_transcoder.wasm` at the site root. Different casing / different subdirectory silently fails texture decode.
- **`cameraTarget` must be a live reference.** Passing `new THREE.Vector3().copy(controls.target)` gives a stale target — orbit distance calculations will drift.
- **`vrmAnimations.asset` is a URL string here** (unlike the PlayCanvas binding, which requires a `pc.Asset`). Don't wrap it.
- **Animation calls before initial load are no-ops.** Guard `playAnimation` / `setVrmLookAtTarget` calls behind `EVENT_MODEL_LOAD`.
- **`triangleBudget` is global.** All streamed models compete via `qualityPriority`.
- **`mobileTriangleBudget: 0` means "fall back to desktop budget"**, not "no limit". Use a large value or `-1` on `iOSMemoryLimit` for truly unlimited on iOS.
- **`.xrg` auto-append.** Any URL not ending in `.xrg` gets `model.xrg` appended. That's usually what you want when pasting a Viverse asset ID URL, but can bite you if you point at a literal file with the wrong extension.
- **Environment maps are `THREE.Texture` here.** Cube-textures work; equirectangular textures should be converted first via `PMREMGenerator`.

## References

- Public docs — Polygon Streaming JavaScript SDK (Three.js): <https://docs.viverse.com/polygon-streaming/polygon-streaming-javascript-sdk/three.js>
- Repo: `web-polygon-streaming/packages/threejs`
- Example app: `web-polygon-streaming/packages/threejs/example`
- Model console: <https://stream.viverse.com/console>
