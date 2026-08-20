---
name: playcanvas-polygon-streaming-html-scripting
description: Option 3 of the Polygon Streaming PlayCanvas SDK. Integrating Polygon Streaming into a PlayCanvas-built HTML page by adding PolygonStreaming.js from the Viverse CDN and creating the streamController / streamableModel entities in JavaScript.
prerequisites: [PlayCanvas project downloaded/built to HTML, ability to edit index.html, Polygon Streaming asset uploaded at stream.viverse.com/console]
tags: [polygon-streaming, viverse, playcanvas, html-scripting, cdn, script-tag, downloaded-build]
---

# PlayCanvas Polygon Streaming — Option 3: HTML Scripting

Use this skill when you have a **PlayCanvas project that will NOT be published to VIVERSE**, and you want to add Polygon Streaming by editing the **downloaded HTML build** and pulling `PolygonStreaming.js` from the Viverse CDN directly. This is **Option 3** in the official *Polygon Streaming PlayCanvas SDK* docs.

Compare with the other PlayCanvas options:

- [`playcanvas-polygon-streaming-viverse-extension`](../playcanvas-polygon-streaming-viverse-extension/SKILL.md) — Option 1, for VIVERSE-published projects.
- [`playcanvas-polygon-streaming-standalone-plugin`](../playcanvas-polygon-streaming-standalone-plugin/SKILL.md) — Option 2, upload `polygon-streaming.js` into the PlayCanvas Editor as a script asset.
- [`playcanvas-polygon-streaming-npm-package`](../playcanvas-polygon-streaming-npm-package/SKILL.md) — JavaScript SDK NPM path with a bundler.

## When To Use This Skill

- Project is **not** going to be published to VIVERSE.
- You've already downloaded a PlayCanvas build (or self-host `playcanvas.js` + your scene) and can edit its `index.html`.
- You want zero NPM tooling and zero editor asset upload — just a `<script>` tag from the Viverse CDN.

## Preflight

- [ ] Downloaded PlayCanvas build available locally, or a self-hosted plain HTML page that already boots a `pc.Application`.
- [ ] Source model uploaded to <https://stream.viverse.com/console> and you have its **Asset ID URL** (⋯ menu → **Copy asset ID**).
- [ ] The site is served with a real HTTP server (service worker + WASM texture transcoder require it — `file://` will not work).

## Installation — Add the CDN Script

Add the Polygon Streaming SDK to your HTML with a versioned CDN URL. Note the filename is **`PolygonStreaming.js`** (capitalized), which is different from Option 2's `polygon-streaming.js` script asset:

```html
<!-- Replace {LATEST_STABLE_VERSION} with a pinned version, e.g. 2.9.2 -->
<script src="https://stream-stage.viverse.com/assets/streamablemodel/{LATEST_STABLE_VERSION}/PolygonStreaming.js"></script>
```

> [!CAUTION]
> Pin the `{LATEST_STABLE_VERSION}` to a specific published release. Do **not** leave it as a literal placeholder in production, and do not blindly point at "latest" — the API is not guaranteed stable across major versions.

## Where To Place The Script

Insert it into the downloaded PlayCanvas `index.html` **after** the PlayCanvas engine `<script>` tag and **after `pc.Application` has been created**, but **before you `.start()` the app and before you attach the `streamController` / `streamableModel` script components**.

## Minimum Working Setup

```javascript
// 1. Add a camera entity for Polygon Streaming SDK to use.
const camera = new pc.Entity('camera');
camera.addComponent('camera', {
  farClip: 1000,
  nearClip: 0.1
});
app.root.addChild(camera);

// 2. Create the Stream Controller entity.
//    Only `camera` is strictly required; all other attributes below are shown for reference.
const streamController = new pc.Entity();
streamController.addComponent('script');
streamController.script.create('streamController', {
  attributes: {
    camera: camera,
    cameraType: 'nonPlayer',
    occlusionCulling: true,
    occlusionGeometry: 'boundingBox',
    occlusionQueryFrequency: 8,
    triangleBudget: 3000000,
    mobileTriangleBudget: 1000000,
    minimumDistance: 0.01,
    distanceFactor: 1.1,
    distanceType: 'boundingBoxCenter',
    maximumQuality: 15000,
    closeUpDistance: 3,
    closeUpDistanceFactor: 5,
    iosMemoryLimit: 0
  }
});

// 3. Create one Streamable Model entity per model.
//    Only `path` is strictly required.
const streamableModel = new pc.Entity();
streamableModel.addComponent('script');
streamableModel.script.create('streamableModel', {
  attributes: {
    path: '/model.xrg',
    qualityPriority: 1,
    useAlpha: true,
    castShadows: true,
    receiveShadows: true,
    doubleSidedMaterials: false,
    initialTrianglePercent: 0.1,
    playAnimationAutomatically: true,
    animation: 0
  }
});

// 4. REQUIRED: streamable model must be a child of the stream controller.
streamController.addChild(streamableModel);

// 5. Add the stream controller to the scene root.
app.root.addChild(streamController);
```

> [!CAUTION]
> The `streamableModel` entity **must be a child of the `streamController`** entity. A top-level streamable model is silently ignored. This is the single most common mistake.

## Attribute Naming Notes — HTML Scripting vs. Other Options

The HTML Scripting doc uses a few attribute names that differ from the other bindings. All of these are aliases the runtime accepts; you can use either but the HTML Scripting doc canonicalizes:

| HTML Scripting (Option 3) | Elsewhere (Options 1, 2 UI labels / JS SDK docs) | Notes |
| --- | --- | --- |
| `iosMemoryLimit` | `iOSMemoryLimit` | Same attribute, different casing. |
| `doubleSidedMaterials` | `forceDoubleSided` | Same attribute. |
| `distanceType: 'boundingBoxCenter'` | *(not exposed in UI)* | Only surfaced in HTML Scripting docs. Controls what point on the mesh is used for camera-distance calculations. |

## Getting a Model URL

1. Upload the source model at <https://stream.viverse.com/console>.
2. **Models** → click ⋯ → **Copy asset ID**.
3. Paste as the `path` attribute on the `streamableModel` component. If the URL doesn't end in `.xrg`, `model.xrg` is appended automatically.

## Stream Controller Attributes (reference)

| Attribute | Type | Default | Notes |
| --- | --- | --- | --- |
| `camera` | `pc.Entity` | — | **Required.** An entity that has a `camera` component. |
| `cameraType` | `'nonPlayer' \| 'player'` | `'nonPlayer'` | |
| `occlusionCulling` | `boolean` | `false` | Requires WebGL 2. |
| `occlusionGeometry` | `'boundingBox' \| 'mesh'` | `'boundingBox'` | Mesh is more accurate but slower. |
| `occlusionQueryFrequency` | `number` | `8` | Queries per second; `0` = every frame. |
| `triangleBudget` | `number` | `5000000` | Global cap. Recommend ≥30% of source polygon count. |
| `mobileTriangleBudget` | `number` | `3000000` | `0` = fall back to desktop budget. |
| `minimumDistance` | `number` | `0.01` | |
| `distanceFactor` | `number` | `1.1` | >1 favors near, <1 favors far. |
| `distanceType` | `'boundingBoxCenter' \| ...` | `'boundingBoxCenter'` | Reference point for camera-distance calc. |
| `maximumQuality` | `number` | `15000` | `0` = unlimited. |
| `closeUpDistance` | `number` | `3` | `0` disables close-up factor. |
| `closeUpDistanceFactor` | `number` | `5` | Should be > `distanceFactor`. |
| `iosMemoryLimit` | `number` (MB) | `0` | `0` = auto, `-1` = no limit. |

## Streamable Model Attributes (reference)

| Attribute | Type | Default | Notes |
| --- | --- | --- | --- |
| `path` | `string` | — | **Required.** URL of the `.xrg` or Viverse asset ID URL. |
| `qualityPriority` | `number` | `1` | Ratio vs. other streamed models. |
| `initialTrianglePercent` | `number` | `0.1` | 0..1. |
| `castShadows` | `boolean` | `true` | |
| `receiveShadows` | `boolean` | `true` | |
| `doubleSidedMaterials` | `boolean` | `false` | Alias of `forceDoubleSided`. |
| `useAlpha` | `boolean` | `true` | |
| `useEmbeddedCollider` | `boolean` | `true` | No-op if the `.xrg` was published without a collider. |
| `playAnimationAutomatically` | `boolean` | `true` | |
| `animation` | `string \| number` | `null` | Name or index (0 = first). |
| `animationStateGraph` | `pc.Asset` | `null` | Anim state graph asset. |
| `animationStateMappings` | `Array<{ state, animation, layer? }>` | `[]` | Required if using a state graph. |
| `vrmAnimations` | `Array<{ name, asset, loop?, default? }>` | `[]` | `asset` is a `pc.Asset` container. |
| `environmentAsset` | `pc.Asset` | `null` | Cubemap for reflections. |
| `hashCode` | `string` | `''` | Optional integrity hash. |

## Playing Animations

Same event API as the other PlayCanvas paths:

```javascript
// Embedded or VRM animation. arg 2 = name or index. arg 3 = transition duration (s).
streamableModel.fire('streamable-model:play-animation', 'idle', 0.2);

// VRM expression.
streamableModel.fire('vrm-expression:start-emotion', 'happy', {
  times:  [0, 0.1, 0.2],
  values: [0, 1,   0]
});
```

## Files Required at the Web Root

Polygon Streaming's chunk cache is a service worker. In production, serve:

```
/service-worker.js
```

Grab it from the same CDN release directory as `PolygonStreaming.js`, or from `packages/core/dist` in the source repo. Without it, models still load but nothing is cached across reloads.

## Verification Checklist

- [ ] `PolygonStreaming.js` CDN URL uses a **pinned version**, not a placeholder or "latest".
- [ ] The script tag appears **after** the PlayCanvas engine and **after** `pc.Application` is created, and **before** `app.start()` and the script-component attachments.
- [ ] `pc.script.create('streamController', ...)` does **not** throw "script not found".
- [ ] `streamableModel` is a **child** of `streamController` in the entity hierarchy.
- [ ] `camera` attribute on the controller refers to an entity that has a `camera` component.
- [ ] `/service-worker.js` is served at the web root in production.
- [ ] The page is served over HTTP(S), not `file://`.

## Gotchas

- **Filename difference.** Option 3 uses **`PolygonStreaming.js`** (from the CDN). Option 2 uses **`polygon-streaming.js`** (uploaded as a PlayCanvas Editor script asset). They are the same runtime; different distribution.
- **Child-of-controller is required.** Silent no-op if you skip it.
- **`iosMemoryLimit`** here, but `iOSMemoryLimit` in the VIVERSE UI / JS SDK docs — both work at runtime, but stay consistent within a project.
- **`doubleSidedMaterials`** here vs. **`forceDoubleSided`** elsewhere — same behavior.
- **`mobileTriangleBudget: 0` means "use desktop budget"**, not "no limit".
- **No bundler = no automatic service worker copy.** You must serve `service-worker.js` manually at the site root for caching to work in production.
- **Don't mix with Option 2.** If you also uploaded `polygon-streaming.js` as a PlayCanvas script asset, the script components will be defined twice and conflict.

## References

- Official docs — Option 3: HTML Scripting: <https://docs.viverse.com/polygon-streaming/integrations-and-plugins/polygon-streaming-playcanvas-sdk/plugin-usage/option-3-html-scripting>
- Official docs — Component Attributes: <https://docs.viverse.com/polygon-streaming/integrations-and-plugins/polygon-streaming-playcanvas-sdk/plugin-usage/component-attributes>
- Model console: <https://stream.viverse.com/console>
- Sister skills:
  - [`playcanvas-polygon-streaming-viverse-extension`](../playcanvas-polygon-streaming-viverse-extension/SKILL.md) — Option 1
  - [`playcanvas-polygon-streaming-standalone-plugin`](../playcanvas-polygon-streaming-standalone-plugin/SKILL.md) — Option 2
  - [`playcanvas-polygon-streaming-npm-package`](../playcanvas-polygon-streaming-npm-package/SKILL.md) — JavaScript SDK NPM
