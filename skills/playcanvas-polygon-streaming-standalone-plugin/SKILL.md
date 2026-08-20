---
name: playcanvas-polygon-streaming-standalone-plugin
description: Option 2 of the Polygon Streaming PlayCanvas SDK. Integrating Polygon Streaming via the Standalone Plugin — upload polygon-streaming.js into the PlayCanvas Editor as a script asset. For projects NOT publishing to VIVERSE and not using a bundler.
prerequisites: [PlayCanvas 1 or 2 loaded via script tag or Editor, ability to upload polygon-streaming.js as a script asset, Basis Library imported in project settings]
tags: [polygon-streaming, viverse, playcanvas, standalone-plugin, downloadable, playcanvas-editor, umd]
---

# PlayCanvas Polygon Streaming — Option 2: Standalone Plugin

Use this skill when you want a **single self-contained JS file** for Polygon Streaming inside the **PlayCanvas Editor** and the project will **not** be published to VIVERSE. This is **Option 2** in the official *Polygon Streaming PlayCanvas SDK* docs and is what the repo README calls the "Downloadable Version".

Compare with the other PlayCanvas options:

- [`playcanvas-polygon-streaming-viverse-extension`](../playcanvas-polygon-streaming-viverse-extension/SKILL.md) — Option 1, for VIVERSE-published projects.
- [`playcanvas-polygon-streaming-html-scripting`](../playcanvas-polygon-streaming-html-scripting/SKILL.md) — Option 3, add `PolygonStreaming.js` from the CDN to a downloaded HTML build.
- [`playcanvas-polygon-streaming-npm-package`](../playcanvas-polygon-streaming-npm-package/SKILL.md) — JavaScript SDK NPM path with a bundler.

## When To Use This Skill

- Integrating Polygon Streaming inside the **PlayCanvas Editor** without the VIVERSE Chrome Browser Extension.
- Prototyping in a static HTML page with `<script>` tags only.
- Shipping a lightweight demo where installing an NPM package is overkill.

## What The Standalone Plugin Is

A single UMD JavaScript file. Two file-name conventions you will see:

| Filename | Where it appears |
| --- | --- |
| **`polygon-streaming.js`** | Official name in the *Polygon Streaming PlayCanvas SDK — Option 2: Standalone Plugin* doc and in the downloadable ZIP from stream.viverse.com. **Use this in the PlayCanvas Editor.** |
| `polygon-streaming.umd.js` | Internal repo build output (`packages/playcanvas/dist/downloadable/dist/polygon-streaming.umd.js`) before it's renamed for distribution. |

When loaded, the UMD bundle registers itself onto the global `pc` namespace and auto-registers the `streamController` and `streamableModel` script components. There is no ESM `import`.

## Getting the File

1. **Download the release ZIP** (official Viverse distribution):

   ```
   https://stream.viverse.com/assets/streamablemodel/<VERSION>/playcanvas/PolygonStreaming-PlayCanvas-<VERSION>.zip
   ```

   Example: `https://stream.viverse.com/assets/streamablemodel/2.9.1/playcanvas/PolygonStreaming-PlayCanvas-2.9.1.zip`. Extract and use `polygon-streaming.js`.

2. **Prebuilt CDN**:

   ```
   https://stream-stage.viverse.com/assets/streamablemodel/<VERSION>/playcanvas/polygon-streaming.js
   ```

   Replace `<VERSION>` with e.g. `2.9.2`.

3. **Build locally** from the source repo:

   ```shell
   # Production (minified)
   scripts/build.ps1 downloadable
   # Development (non-minified)
   scripts/build.ps1 downloadable-dev
   ```

   Output: `packages/playcanvas/dist/downloadable/dist/polygon-streaming.umd.js`. Rename to `polygon-streaming.js` when uploading into the Editor.

## Preflight

- [ ] PlayCanvas engine is loaded **before** `polygon-streaming.js`.
- [ ] **Basis Library imported**: in the PlayCanvas Editor go to **Settings → Rendering → IMPORT BASIS**. Polygon Streaming's KTX2 textures require this.
- [ ] Target PlayCanvas version known (standalone works with `1 - 2`; the preview app pins `2.11.8`).
- [ ] For the Editor path: upload access to the project.

> [!CAUTION]
> **The Basis Library import is easy to miss.** Without it, KTX2 textures fail to decode and streamed meshes render either untextured or invisible. This is the most common Option 2 integration bug.

## Integration — PlayCanvas Editor (recommended)

1. **Assets → Upload** and drop `polygon-streaming.js`.
2. Select the uploaded script asset and press **Parse**. Confirm there are no errors — this is how the editor discovers the `streamController` and `streamableModel` script components.
3. In your scene, create a **Camera** entity with a `camera` component.
4. Create a **StreamController** entity. Add a `script` component with the `streamController` script. In the inspector, drag your camera entity into the `Camera` attribute. Set `Triangle Budget`, `Camera Type`, etc.
5. Create a **StreamableModel** entity **as a child of the StreamController entity**. Add a `script` component with the `streamableModel` script. Paste your model URL (from [stream.viverse.com/console](https://stream.viverse.com/console) → ⋯ → **Copy asset ID**) into the `Path or url to model` attribute.
6. Press **Launch** to preview.

> [!CAUTION]
> If you re-upload a new version of `polygon-streaming.js`, press **Parse** again. The Editor caches the attribute schema.

## Integration — Plain HTML

```html
<!doctype html>
<html>
  <head>
    <!-- 1. PlayCanvas engine first -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/playcanvas/2.11.8/playcanvas.js"></script>
    <!-- 2. Polygon Streaming standalone (registers script components on load) -->
    <script src="https://stream-stage.viverse.com/assets/streamablemodel/2.9.2/playcanvas/polygon-streaming.js"></script>
  </head>
  <body>
    <canvas id="application"></canvas>
    <script>
      const canvas = document.getElementById('application');
      const app = new pc.Application(canvas, {});
      app.setCanvasFillMode(pc.FILLMODE_FILL_WINDOW);
      app.setCanvasResolution(pc.RESOLUTION_AUTO);
      app.start();

      const camera = new pc.Entity('camera');
      camera.addComponent('camera');
      camera.setPosition(0, 2, 8);
      app.root.addChild(camera);

      const streamController = new pc.Entity('StreamController');
      streamController.addComponent('script');
      streamController.script.create('streamController', {
        attributes: { camera, triangleBudget: 5000000 }
      });

      const model = new pc.Entity('StreamableModel');
      model.addComponent('script');
      model.script.create('streamableModel', {
        attributes: { path: 'https://stream.viverse.com/demos/jet-engine-11m/' }
      });

      streamController.addChild(model);   // REQUIRED: child of controller
      app.root.addChild(streamController);
    </script>
  </body>
</html>
```

> [!CAUTION]
> Load **PlayCanvas first, Polygon Streaming second**. Reversing the order means the UMD bundle can't find `pc` and silently fails to register.

## Playing Animations

The UMD build exposes the exact same script components as the NPM package:

```javascript
// Embedded or VRM animation. arg 2 = name or index. arg 3 = transition seconds.
streamableModelEntity.fire('streamable-model:play-animation', 'idle', 0.2);

// VRM expression
streamableModelEntity.fire('vrm-expression:start-emotion', 'happy', {
  times:  [0, 0.1, 0.2],
  values: [0, 1,   0]
});

// State-graph-triggered transition
streamableModelEntity.anim.setBoolean('running', true);
```

For VRM `.vrma` animations, set the `vrmAnimations` attribute at create time. In the Editor use the inspector list of `{ name, asset, loop, default }`; in code pass `pc.Asset` container instances.

> [!CAUTION]
> Do not access `entity.anim` in a script's `initialize()` method. The anim component is only added after the streamable model has loaded.

## Priority Level presets

The Editor inspector exposes a `Priority Level` dropdown that writes into `qualityPriority`:

| Priority Level | `qualityPriority` value |
| --- | --- |
| Default | `1` |
| Higher | `1.5` |
| Highest | `2` |
| Custom | User-entered number |

## Attributes

The attribute set is identical to the NPM package. See [`playcanvas-polygon-streaming-npm-package`](../playcanvas-polygon-streaming-npm-package/SKILL.md#stream-controller-attributes-reference) for the full reference. Highlights:

- `streamController`: `camera` (required), `cameraType`, `triangleBudget`, `mobileTriangleBudget`, `distanceFactor`, `distanceType`, `maximumQuality`, `iOSMemoryLimit`, `occlusionCulling`.
- `streamableModel`: `path` (required), `qualityPriority`, `initialTrianglePercent`, `castShadows`, `receiveShadows`, `useAlpha`, `useEmbeddedCollider`, `animation`, `vrmAnimations`, `environmentAsset`.

Attribute aliases you may see:

- `iOSMemoryLimit` (canonical, JS SDK docs) === `iosMemoryLimit` (HTML Scripting docs).
- `forceDoubleSided` (canonical) === `doubleSidedMaterials` (HTML Scripting docs).

## Files Required at the Web Root

Same as the NPM path: the service worker must be at `/service-worker.js`.

- In the **PlayCanvas Editor Launch preview**, PlayCanvas hosts the page and no service worker is present — chunk caching is disabled. Models still load; they aren't cached across reloads. Don't diagnose this as a perf regression.
- For a **published** build, add `service-worker.js` to your published site's root. Grab it from `.../<VERSION>/playcanvas/service-worker.js` or from `packages/core/dist` in the repo.

## Query Parameters

The UMD build honors the same URL params as the NPM path:

- `md`, `df`, `mq`, `cd`, `cdf`
- `oc`, `og` (`mesh` | `boundingBox`), `of`
- `iosml`, `rhn`
- `webgpu=true`
- `tlp` (`off` | `coverage`), `tls`
- `model-only=true`
- `minimize-cpu=true`
- `v` / `version` — pick a specific published player version (staging/prod player only)
- `pc` — pick a specific PlayCanvas engine version (staging/prod player only)

## Verification Checklist

- [ ] Basis Library imported via **Settings → Rendering → IMPORT BASIS**.
- [ ] PlayCanvas `<script>` tag comes **before** `polygon-streaming.js`.
- [ ] After the UMD is loaded, `pc.script.create('streamController', ...)` does **not** throw "script not found".
- [ ] `StreamableModel` entity is a **child** of the `StreamController` entity.
- [ ] In the Editor: **Parse** was pressed after upload; script attributes appear in the inspector.
- [ ] For a published site: `service-worker.js` is served from `/`.

## Gotchas

- **Missing Basis import.** Textures fail silently.
- **Script load order.** PlayCanvas must exist as global `pc` before the UMD file runs.
- **Editor caching after re-upload.** Press **Parse** again after every re-upload.
- **`streamableModel` must be a child of `streamController`.** A top-level streamable model is silently ignored.
- **Editor Launch has no service worker.** Cold-cache loads on every Launch are expected.
- **PlayCanvas version drift.** Shader-chunk APIs assume 2.6+ paths.
- **Don't mix Option 2 + NPM (JS SDK) in one page.** Duplicate script-component registration will conflict.
- **Don't mix Option 2 + Option 3.** Same underlying components; only load one.
- **Don't rename the uploaded script asset.** Internal script names (`streamController`, `streamableModel`) are independent of file name; keep the file name conventional.
- **`downloadable-dev` is non-minified.** Do not ship to production.

## References

- Official docs — Option 2: Standalone Plugin: <https://docs.viverse.com/polygon-streaming/integrations-and-plugins/polygon-streaming-playcanvas-sdk/plugin-usage/option-2-standalone-plugin>
- Official docs — Component Attributes: <https://docs.viverse.com/polygon-streaming/integrations-and-plugins/polygon-streaming-playcanvas-sdk/plugin-usage/component-attributes>
- Repo build script: `scripts/build.ps1 downloadable` / `downloadable-dev`
- Model console: <https://stream.viverse.com/console>
- Sister skills:
  - [`playcanvas-polygon-streaming-viverse-extension`](../playcanvas-polygon-streaming-viverse-extension/SKILL.md) — Option 1
  - [`playcanvas-polygon-streaming-html-scripting`](../playcanvas-polygon-streaming-html-scripting/SKILL.md) — Option 3
  - [`playcanvas-polygon-streaming-npm-package`](../playcanvas-polygon-streaming-npm-package/SKILL.md) — JavaScript SDK NPM
