---
name: babylonjs-polygon-streaming
description: Integrating Viverse Polygon Streaming (.xrg) into a Babylon.js scene via the @polygon-streaming/web-player-babylonjs NPM package. NOT covered by the official Viverse public documentation — this skill is derived from the source repo (packages/babylonjs) and may lag or diverge from official releases.
prerequisites: [Babylon.js scene with camera + engine + render loop, @babylonjs/loaders/glTF, optional Ammo.js for collider, service worker at web root]
tags: [polygon-streaming, viverse, babylonjs, xrg, streaming, ammo, unofficial]
---

# Polygon Streaming — Babylon.js (Unofficial / Repo-only)

> [!CAUTION]
> **This binding is NOT documented in the official Viverse public docs.** As of the current *Polygon Streaming JavaScript SDK* documentation set, only **PlayCanvas** and **Three.js** are listed. Everything below is derived from the source repo (`web-polygon-streaming/packages/babylonjs`) and its example app. API surface, package name, and behavior may lag or diverge from official Viverse releases. Prefer the Three.js or PlayCanvas bindings when possible; use this only if a Babylon.js integration is required.

Use this skill when integrating Viverse Polygon Streaming into a Babylon.js scene via `@polygon-streaming/web-player-babylonjs`. The binding is an ES-module package driven by a per-frame `streamController.update()` call inside `engine.runRenderLoop`.

## When To Use This Skill

- Loading `.xrg` streamed models into a Babylon.js scene.
- Using the embedded collider from the streamed model (requires Ammo.js).

## Preflight

- [ ] A Babylon.js scene with `Engine`, `Scene`, a `Camera`, and a render loop.
- [ ] `@babylonjs/loaders/glTF` is imported (Polygon Streaming reuses the glTF loader for internal assets).
- [ ] Service worker at `/service-worker.js`.
- [ ] If you want the embedded collider: Ammo.js is loaded (Ammo is the **only** supported physics backend because it is the only one with concave-collider support).

## Package

```json
"dependencies": {
  "@polygon-streaming/web-player-babylonjs": "latest",
  "@babylonjs/core": "^7",
  "@babylonjs/loaders": "^7"
}
```

## Minimum Working Setup

```javascript
import * as BABYLON from '@babylonjs/core/Legacy/legacy';
import '@babylonjs/loaders/glTF';
import {
  StreamController,
  loadWasmModule
} from '@polygon-streaming/web-player-babylonjs';

// (Optional) Ammo.js — only needed for the embedded collider.
import ammoWasmJsUrl   from './lib/ammo.wasm.js?url';
import ammoWasmWasmUrl from './lib/ammo.wasm.wasm?url';
import ammoJsUrl       from './lib/ammo.js?url';

loadWasmModule('Ammo', ammoWasmJsUrl, ammoWasmWasmUrl, ammoJsUrl)
  .then(ammoInstance => {
    // ...create engine, scene, camera, cameraTarget as usual...

    const streamController = new StreamController(
      camera,
      engine,
      scene,
      cameraTarget,     // BABYLON.Vector3
      {
        cameraType: 'nonPlayer',
        triangleBudget: 5_000_000,
        mobileTriangleBudget: 3_000_000,
        ammoInstance    // required for `useEmbeddedCollider`
      }
    );

    const modelParent = new BABYLON.TransformNode('Model parent', scene);
    modelParent.position.set(0, 1, 0);

    streamController.addModel(
      'https://stream.viverse.com/demos/jet-engine-11m/',
      modelParent,
      { qualityPriority: 1 }
    );

    engine.runRenderLoop(() => {
      scene.render();
      streamController.update();   // REQUIRED every frame
    });
  });
```

> [!CAUTION]
> `streamController.update()` **must** run every frame inside the render loop. Without it, streaming stops after the initial chunk.

## Ammo.js — Only for the Embedded Collider

If your model doesn't use `useEmbeddedCollider`, you can skip Ammo entirely. When you do need it:

- Only **Ammo.js** is supported. Cannon, Havok, and Oimo do not support concave meshes.
- Use the **v1 physics API**: add `PhysicsImpostor` to your meshes, **not** `PhysicsAggregate`/`PhysicsBody` (which are v2-only).
- Load Ammo through the package's `loadWasmModule` helper so the wasm/glue URLs are correctly wired.
- Pass the resolved `ammoInstance` to the `StreamController` options.

> [!CAUTION]
> Passing a `PhysicsAggregate`-style setup expecting the embedded collider to work will silently fail — Polygon Streaming attaches impostors, not bodies.

## Getting a Model URL

1. Upload the source model at [stream.viverse.com/console](https://stream.viverse.com/console).
2. In **Models**, click ⋯ → **Copy asset ID**.
3. Paste as the first argument to `streamController.addModel(...)`.

If the URL doesn't end in `.xrg`, `model.xrg` is appended automatically.

## Stream Controller Parameters

Constructor: `new StreamController(camera, engine, scene, cameraTarget, options)`

| Required arg | Type | Notes |
| --- | --- | --- |
| `camera` | `BABYLON.Camera` | The scene camera. |
| `engine` | `BABYLON.Engine` | The engine. |
| `scene` | `BABYLON.Scene` | The root scene. |
| `cameraTarget` | `BABYLON.Vector3` | The point the camera is looking at. |

`options`:

| Option | Type | Default | Notes |
| --- | --- | --- | --- |
| `cameraType` | `'nonPlayer' \| 'player'` | `'nonPlayer'` | |
| `triangleBudget` | `number` | `5_000_000` | Global triangle cap. |
| `mobileTriangleBudget` | `number` | `3_000_000` | `0` = use desktop budget. |
| `minimumDistance` | `number` | `0.01` | |
| `distanceFactor` | `number` | `1.1` | |
| `maximumQuality` | `number` | `15000` | `0` = unlimited. |
| `closeUpDistance` | `number` | `3` | `0` disables close-up factor. |
| `closeUpDistanceFactor` | `number` | `5` | Should be > `distanceFactor`. |
| `iOSMemoryLimit` | `number` (MB) | `0` | `0` = auto, `-1` = no limit. |
| `showLoadingModel` | `boolean` | `true` | |
| `loadingModelUrl` | `string` | `null` | Override loading GLB. |
| `ammoInstance` | `object` | `null` | Required for embedded colliders. |

## Streamable Model Parameters

Method: `streamController.addModel(url, modelParent, options)`

| Required arg | Type | Notes |
| --- | --- | --- |
| `url` | `string` | URL of the `.xrg` (auto-appended if missing). |
| `modelParent` | `BABYLON.TransformNode` | Parent for streamed meshes. |

`options`:

| Option | Type | Default | Notes |
| --- | --- | --- | --- |
| `qualityPriority` | `number` | `1` | |
| `initialTrianglePercent` | `number` | `0.1` | |
| `castShadows` | `boolean` | `true` | |
| `receiveShadows` | `boolean` | `true` | |
| `forceDoubleSided` | `boolean` | `false` | |
| `useAlpha` | `boolean` | `true` | |
| `environmentMap` | `BABYLON.CubeTexture` | `null` | |
| `hashCode` | `string` | `''` | |

> [!CAUTION]
> The Babylon.js binding does **not** currently expose the animation-triggering API (`playAnimation`, `vrmAnimations`, VRM expressions, VRM look-at) that the Three.js and PlayCanvas bindings do. If you need VRM animation on a Babylon scene, use the Three.js or PlayCanvas binding instead.

## Files Required at the Web Root

- `/service-worker.js` — chunk caching for `.xrg` fetches. Copy from `@polygon-streaming/web-player-core` during your build.
- If Ammo is enabled: your bundler needs to serve `ammo.wasm.js`, `ammo.wasm.wasm`, and `ammo.js`. The example uses `?url` imports so Vite fingerprints them; adapt for your bundler.

## Verification Checklist

- [ ] `streamController.update()` runs every frame inside `engine.runRenderLoop`.
- [ ] `modelParent` is a `TransformNode` attached to the scene before `addModel`.
- [ ] `@babylonjs/loaders/glTF` was imported (side-effect registration).
- [ ] `/service-worker.js` is at the web root in production.
- [ ] If `useEmbeddedCollider`: Ammo is loaded via `loadWasmModule` and `ammoInstance` is passed to the controller.

## Gotchas

- **Missing `update()` call.** Mesh loads at low quality once and never refines. Symptom identical to the Three.js case.
- **Wrong physics backend.** Only Ammo.js works. Havok/Cannon/Oimo cannot handle the concave colliders Polygon Streaming produces.
- **Physics v1 API only.** `PhysicsImpostor`, not `PhysicsAggregate`/`PhysicsBody`.
- **Legacy import.** The examples import from `@babylonjs/core/Legacy/legacy` to grab the full namespace. If your project uses tree-shaken side-effect imports (`@babylonjs/core`), you must import the specific classes you use (`Engine`, `Scene`, `TransformNode`, `Vector3`, `CubeTexture`, etc.) yourself; the binding does not do this for you.
- **No animation API.** VRM/embedded animation, VRM expressions, and VRM look-at are not exposed for Babylon.js. Streamed models render statically (or with their embedded rest pose).
- **`triangleBudget` is global.** Multiple streamed models compete via `qualityPriority`.
- **`mobileTriangleBudget: 0` means "fall back to desktop budget"**, not "no limit".
- **`.xrg` auto-append.** URLs without `.xrg` get `model.xrg` appended. Usually correct when pasting Viverse asset URLs.

## References

- Repo: `web-polygon-streaming/packages/babylonjs` (source of truth — no official public docs page)
- Example app: `web-polygon-streaming/packages/babylonjs/example`
- Public docs (PlayCanvas + Three.js only, Babylon.js not listed): <https://docs.viverse.com/polygon-streaming/polygon-streaming-javascript-sdk>
- Model console: <https://stream.viverse.com/console>
