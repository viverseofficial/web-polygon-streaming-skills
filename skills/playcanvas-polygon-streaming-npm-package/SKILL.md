---
name: playcanvas-polygon-streaming-npm-package
description: Polygon Streaming JavaScript SDK — PlayCanvas binding. Integrating Viverse Polygon Streaming (.xrg) into a PlayCanvas app via the @polygon-streaming/web-player-playcanvas NPM package (ES module).
prerequisites: [PlayCanvas 1 or 2, ES-module bundler (Vite/Webpack), service worker served at web root]
tags: [polygon-streaming, viverse, playcanvas, xrg, streaming, npm, esm, javascript-sdk]
---

# Polygon Streaming JavaScript SDK — PlayCanvas (NPM Package)

Use this skill when integrating Polygon Streaming into a PlayCanvas application that has a bundler and imports `@polygon-streaming/web-player-playcanvas` as an ES module. This is the **Polygon Streaming JavaScript SDK → PlayCanvas** path in the official docs, used by Viverse Create (formerly Viverse World), the PlayCanvas editor extension, and any custom PlayCanvas app built with Vite/Webpack.

If you want a different integration path, see:

- [`playcanvas-polygon-streaming-viverse-extension`](../playcanvas-polygon-streaming-viverse-extension/SKILL.md) — Option 1 of the PlayCanvas SDK (VIVERSE-published projects).
- [`playcanvas-polygon-streaming-standalone-plugin`](../playcanvas-polygon-streaming-standalone-plugin/SKILL.md) — Option 2 (UMD `polygon-streaming.js` uploaded into the Editor, no bundler).
- [`playcanvas-polygon-streaming-html-scripting`](../playcanvas-polygon-streaming-html-scripting/SKILL.md) — Option 3 (add `PolygonStreaming.js` from the CDN to a downloaded HTML build).

## When To Use This Skill

- You are adding a streamed `.xrg` model to an ES-module-based PlayCanvas project.
- You need to know the exact `streamController` / `streamableModel` script component attributes.
- You want to trigger embedded animations, VRM animations, or VRM expressions.

## Preflight

- [ ] `playcanvas` version 1 or 2 is available (peer dependency).
- [ ] Bundler serves a service worker at `/service-worker.js` (see "Files Required" below).
- [ ] For internal Viverse builds, `.npmrc` has the internal registry token; for the public build, install `@polygon-streaming/web-player-playcanvas` from npmjs.com.

## Package

```json
"dependencies": {
  "@polygon-streaming/web-player-playcanvas": "^2.9.2",
  "playcanvas": "^2"
}
```

The package's `peerDependencies` accepts `playcanvas` in the range `1 - 2`.

## Minimum Working Setup

Polygon Streaming uses **two PlayCanvas script components**: one `streamController` and one `streamableModel` per model. The `streamableModel` entity **must be a child of the `streamController` entity**.

```javascript
import * as pc from 'playcanvas';
import { registerComponents } from '@polygon-streaming/web-player-playcanvas';

// 1. Register the streamController + streamableModel script components with PlayCanvas.
registerComponents();

// ...create app + canvas as normal...

// 2. Camera the stream controller will use for distance/quality decisions.
const camera = new pc.Entity('camera');
camera.addComponent('camera');
app.root.addChild(camera);

// 3. Stream controller — one per scene.
const streamController = new pc.Entity('Stream Controller');
streamController.addComponent('script');
streamController.script.create('streamController', {
  attributes: {
    camera,
    cameraType: 'nonPlayer',       // or 'player'
    triangleBudget: 5_000_000,
    mobileTriangleBudget: 3_000_000
  }
});

// 4. One streamableModel entity per model, as a child of the controller.
const streamableModel = new pc.Entity('Streaming Model');
streamableModel.addComponent('script');
streamableModel.script.create('streamableModel', {
  attributes: {
    path: 'https://stream.viverse.com/demos/jet-engine-11m/',
    qualityPriority: 1
  }
});

streamController.addChild(streamableModel);   // REQUIRED: child of controller
app.root.addChild(streamController);
```

> [!CAUTION]
> If the `streamableModel` entity is **not a child** of the `streamController` entity, the model will never start streaming. This is the #1 integration mistake.

## Named Exports Available

The most commonly used named exports from `@polygon-streaming/web-player-playcanvas`:

- `registerComponents()` — must be called once before creating any streaming entities.
- `STREAM_CONTROLLER_COMPONENT_NAME` — the script component name (`'streamController'`). Use this constant when Viverse Create / the editor extension programmatically references the component.
- `STREAMING_MODEL_COMPONENT_NAME` — the script component name (`'streamableModel'`).
- `toAnimStateGraphAsset(obj)` — converts a plain JS state-graph object into a `pc.Asset` for the `animationStateGraph` attribute.

## Files Required at the Web Root

Polygon Streaming's fetch caching relies on a service worker served from the site root:

```
/service-worker.js
```

If you use Vite, the `packages/playcanvas/example/vite.config.js` in the repo shows the required `vite-plugin-static-copy` setup. When integrating into an existing app, copy `service-worker.js` from `@polygon-streaming/web-player-core` to your public directory during the build.

> [!CAUTION]
> Without the service worker at the web root, models still load, but chunk caching is disabled and repeat visits refetch everything.

## Playing an Embedded or VRM Animation

Fire the `streamable-model:play-animation` event on the streaming-model entity:

```javascript
// arg 2: animation name (string) OR index (number, 0 = first)
// arg 3: transition duration in seconds (optional, default 1)
streamableModel.fire('streamable-model:play-animation', 'idle', 0.2);
```

For VRM animations (`.vrma`) you must declare them up-front in `vrmAnimations`, because the streamable model needs the container assets at creation time:

```javascript
streamableModel.script.create('streamableModel', {
  attributes: {
    path: '/model.xrg',
    vrmAnimations: [
      {
        name: 'idle',
        asset: new pc.Asset('', 'container', { url: '/Idle.vrma' }),
        default: true            // played when a non-looping animation finishes
      },
      {
        name: 'yawn',
        asset: new pc.Asset('', 'container', { url: '/Yawn.vrma' }),
        loop: false              // one-shot; returns to the default animation after
      }
    ]
  }
});
```

## Animating a VRM Expression

```javascript
streamableModel.fire('vrm-expression:start-emotion', 'happy', {
  times:  [0, 0.1, 0.2],  // seconds
  values: [0, 1,   0]      // expression weights 0..1
});
```

The expression name must exist in the source VRM the `.xrg` was generated from.

## Load Events

```javascript
streamableModel.once('streamable-model:load',
  (boundingBox, _, willUseEmbeddedCollider) => {
    // Model has its first mesh — safe to reframe the camera, add a collider, etc.
  }
);

streamableModel.once('streamable-model:load-error', (error) => {
  console.error(error);
});
```

## Stream Controller Attributes (reference)

| Attribute | Type | Default | Notes |
| --- | --- | --- | --- |
| `camera` | `pc.Entity` | — | Required. Camera used for distance/quality. |
| `cameraType` | `'nonPlayer' \| 'player'` | `'nonPlayer'` | `'player'` if the camera is attached to a first-person controller. |
| `occlusionCulling` | `boolean` | `false` | Requires WebGL 2. |
| `occlusionGeometry` | `'boundingBox' \| 'mesh'` | `'boundingBox'` | `'mesh'` is more accurate but slower. |
| `occlusionQueryFrequency` | `number` | `8` | Queries per second. `0` = every frame. |
| `triangleBudget` | `number` | `5_000_000` | Max simultaneous triangles in the scene. |
| `mobileTriangleBudget` | `number` | `3_000_000` | Used on mobile. `0` = use `triangleBudget`. |
| `minimumDistance` | `number` | `0.01` | Clamp on distance-to-camera. |
| `distanceFactor` | `number` | `1.1` | >1 favors nearby geometry. |
| `maximumQuality` | `number` | `15000` | Stops refining beyond this quality. `0` = unlimited. |
| `closeUpDistance` | `number` | `3` | Distance where close-up factor kicks in. `0` disables. |
| `closeUpDistanceFactor` | `number` | `5` | Should be > `distanceFactor`. |
| `distanceType` | `string` | `'boundingBoxCenter'` | Reference point for camera-distance calculation. Surfaced primarily in HTML Scripting docs. |
| `iOSMemoryLimit` | `number` (MB) | `0` | `0` = auto, `-1` = no limit. Aliased as `iosMemoryLimit` in the HTML Scripting docs. |
| `showLoadingModel` | `boolean` | `true` | Animated loading placeholder. |
| `loadingModelUrl` | `string` | `null` | Custom loading GLB. |

> [!TIP]
> **Rule of thumb**: set `triangleBudget` to at least **30% of the source model's polygon count**. For a 10M-poly source, use ≥ 3,000,000.

## Streamable Model Attributes (reference)

| Attribute | Type | Default | Notes |
| --- | --- | --- | --- |
| `path` | `string` | `'/model.xrg'` | Path or URL of the `.xrg` (or a Viverse asset ID URL). |
| `qualityPriority` | `number` | `1` | Relative quality vs. other streamed models in the scene. See Priority Level presets below. |
| `forceDoubleSided` | `boolean` | `false` | Aliased as `doubleSidedMaterials` in HTML Scripting docs. |
| `initialTrianglePercent` | `number` | `0.1` | Portion of budget to allocate on first load. |
| `castShadows` | `boolean` | `true` | |
| `receiveShadows` | `boolean` | `true` | |
| `useAlpha` | `boolean` | `true` | Disable for opaque-only rendering (faster). |
| `useEmbeddedCollider` | `boolean` | `true` | Uses the collider bundled in the `.xrg`. |
| `playAnimationAutomatically` | `boolean` | `true` | |
| `animation` | `string \| number` | `null` | Name or index of the embedded animation. |
| `animationStateGraph` | `pc.Asset` | `null` | Use `toAnimStateGraphAsset()` to build. |
| `animationStateMappings` | `Array<{ state, animation, layer? }>` | `[]` | Required when using a state graph. |
| `vrmAnimations` | `Array<{ name, asset, loop?, default? }>` | `[]` | See VRM section above. |
| `environmentAsset` | `pc.Asset` (cubemap or texture) | `null` | Prefiltered IBL. |
| `hashCode` | `string` | `''` | Optional integrity hash. |

## Priority Level Presets

The VIVERSE Extension / Editor inspector exposes a `Priority Level` dropdown that writes into `qualityPriority`. When configuring via code you set `qualityPriority` directly:

| Priority Level | `qualityPriority` value |
| --- | --- |
| Default | `1` |
| Higher | `1.5` |
| Highest | `2` |
| Custom | User-entered number |

> [!CAUTION]
> **Animations do not play inside the VIVERSE extension editor preview.** If your project uses the VIVERSE Extension, you must **Publish** the scene to VIVERSE to see the model animate. Static geometry preview works in the editor.

## Getting a Model URL

1. Upload the source model at [stream.viverse.com/console](https://stream.viverse.com/console).
2. In **Models**, click the ⋯ menu → **Copy asset ID**.
3. Paste as the `path` attribute on the `streamableModel` component.

## Query Parameters (useful for testing)

The player also honors many URL query parameters that map to controller attributes:

- `md` (minimum-distance), `df` (distance-factor), `mq` (maximum-quality)
- `cd` (close-up-distance), `cdf` (close-up-distance-factor)
- `oc` (occlusion-culling), `og` (occlusion-geometry: `mesh` | `boundingBox`), `of` (occlusion-frequency)
- `iosml` (ios-memory-limit), `rhn` (remove-hidden-nodes)
- `webgpu=true` — force WebGPU device
- `tlp` (texture-lod-policy: `off` | `coverage`), `tls` (texture-lod-scale, lower = higher quality)
- `model-only=true` — hide background & mini-stats
- `minimize-cpu=true` — pause update loop when idle (default in iframes)

## Verification Checklist

- [ ] `registerComponents()` is called exactly once, before creating any `streamController` / `streamableModel` script components.
- [ ] `streamableModel` entity is a **child** of the `streamController` entity.
- [ ] `camera` attribute on the controller is set to an entity that actually has a `camera` component.
- [ ] `/service-worker.js` is served at the web root in production.
- [ ] On WebGL 1 devices, `occlusionCulling` is disabled or gracefully skipped.
- [ ] For VRM: every entry in `vrmAnimations` uses a `pc.Asset` of type `'container'`.

## Gotchas

- **Child-of-controller is required.** A `streamableModel` at the root of the scene silently does nothing.
- **`registerComponents()` before entity creation.** Calling it after the entities are created leaves them without their script definitions.
- **`triangleBudget` is global.** It's the total across all streamed models — models compete via `qualityPriority`, not by having their own budget.
- **`mobileTriangleBudget: 0` is not "no limit"** — it means "fall back to the desktop `triangleBudget`". Use a very large number if you truly want no mobile cap, or `-1` on `iOSMemoryLimit` for iOS-specific.
- **VRM animations must be declared at create time.** They are container assets, not URLs; wrap them with `new pc.Asset('', 'container', { url: ... })`.
- **`animation` and `animationStateGraph` are mutually exclusive in practice.** If you supply a state graph, drive playback through it and set `playAnimationAutomatically: false` if you want manual control.
- **`useEmbeddedCollider: true` requires the model was published with one.** If the `.xrg` has no collider, this is a no-op.
- **UMD vs ESM.** If you find yourself needing `<script src="polygon-streaming.js">` (e.g. inside the PlayCanvas Editor), you want [`playcanvas-polygon-streaming-standalone-plugin`](../playcanvas-polygon-streaming-standalone-plugin/SKILL.md) (Option 2), not this one. If you're targeting VIVERSE publication with no bundler, use [`playcanvas-polygon-streaming-viverse-extension`](../playcanvas-polygon-streaming-viverse-extension/SKILL.md) (Option 1).

## References

- Public docs — Polygon Streaming JavaScript SDK (PlayCanvas): <https://docs.viverse.com/polygon-streaming/polygon-streaming-javascript-sdk/playcanvas>
- Repo: `web-polygon-streaming/packages/playcanvas`
- Example app: `web-polygon-streaming/packages/playcanvas/example`
- Model console: <https://stream.viverse.com/console>
- Sister skills:
  - [`playcanvas-polygon-streaming-viverse-extension`](../playcanvas-polygon-streaming-viverse-extension/SKILL.md) — Option 1
  - [`playcanvas-polygon-streaming-standalone-plugin`](../playcanvas-polygon-streaming-standalone-plugin/SKILL.md) — Option 2
  - [`playcanvas-polygon-streaming-html-scripting`](../playcanvas-polygon-streaming-html-scripting/SKILL.md) — Option 3
