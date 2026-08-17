---
name: playcanvas-polygon-streaming-standalone
description: Integrating Viverse Polygon Streaming into PlayCanvas via the standalone UMD build (polygon-streaming.umd.js) — no bundler required. Used in the PlayCanvas Editor and plain <script> setups.
prerequisites: [PlayCanvas 1 or 2 loaded via script tag or Editor, ability to upload a script asset]
tags: [polygon-streaming, viverse, playcanvas, umd, standalone, downloadable, playcanvas-editor]
---

# PlayCanvas Polygon Streaming — Standalone (UMD)

Use this skill when you want a **single self-contained JS file** for Polygon Streaming and no bundler. This is the mode the repo README calls the "Downloadable Version" and is what you upload into the **PlayCanvas Editor** when you do not want to use the Viverse Create browser extension.

If you have a bundler (Vite/Webpack) and are `import`-ing from `@polygon-streaming/web-player-playcanvas`, use [`playcanvas-polygon-streaming-npm-package`](../playcanvas-polygon-streaming-npm-package/SKILL.md) instead.

## When To Use This Skill

- Integrating Polygon Streaming inside the **PlayCanvas Editor** without the Viverse Create extension.
- Prototyping in a static HTML page with `<script>` tags only.
- Shipping a lightweight demo where installing an NPM package is overkill.

## What "Standalone" Means Here

The standalone build is a UMD bundle:

```
polygon-streaming.umd.js         # production, minified
polygon-streaming.umd.js         # dev, non-minified (build with -dev)
```

When loaded, it registers itself onto the global `pc` namespace and self-registers its script components. There is **no ESM import**; you just load the file and PlayCanvas can immediately create `streamController` / `streamableModel` script components.

## Getting the File

Two options:

1. **Prebuilt CDN** (production versions published from this repo):

   ```
   https://stream-stage.viverse.com/assets/streamablemodel/<VERSION>/playcanvas/polygon-streaming.js
   ```

   Replace `<VERSION>` with e.g. `2.1.1`. Staging CDN prefix is `https://d2s1xgv6f13wzb.cloudfront.net`, production is `https://d291z28qrimhto.cloudfront.net`.

2. **Build locally** from the repo:

   ```shell
   # Production (minified)
   scripts/build.ps1 downloadable

   # Development (non-minified, debugger-friendly)
   scripts/build.ps1 downloadable-dev
   ```

   Output: `packages/playcanvas/dist/downloadable/dist/polygon-streaming.umd.js`.

## Preflight

- [ ] PlayCanvas engine is loaded **before** `polygon-streaming.umd.js`.
- [ ] You know your target PlayCanvas version (the standalone was tested against `1 - 2`; the preview app pins `2.11.8`).
- [ ] For the PlayCanvas Editor path: you have upload access to the project.

## Integration — Plain HTML

```html
<!doctype html>
<html>
  <head>
    <!-- 1. PlayCanvas engine first -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/playcanvas/2.11.8/playcanvas.js"></script>
    <!-- 2. Polygon Streaming standalone (registers script components on load) -->
    <script src="https://stream-stage.viverse.com/assets/streamablemodel/2.4.0/playcanvas/polygon-streaming.js"></script>
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

      const streamController = new pc.Entity('Stream Controller');
      streamController.addComponent('script');
      streamController.script.create('streamController', {
        attributes: { camera, triangleBudget: 5000000 }
      });

      const model = new pc.Entity('Streaming Model');
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

## Integration — PlayCanvas Editor

1. In the editor: **Assets → Upload** → drop `polygon-streaming.umd.js`.
2. Select the uploaded script asset and press **Parse**. Confirm there are no errors — this is how the editor discovers the `streamController` and `streamableModel` script components.
3. In your scene, create an entity for the camera. Add a **camera** component.
4. Create a **Stream Controller** entity. Add a **script** component with the `streamController` script. In the inspector, drag your camera entity into the `camera` attribute. Set `triangleBudget`, `cameraType`, etc.
5. Create a **Streaming Model** entity **as a child of the Stream Controller entity**. Add a **script** component with the `streamableModel` script. Paste your model URL (from [stream.viverse.com/console](https://stream.viverse.com/console) → ⋯ → **Copy asset ID**) into the `path` attribute.
6. Press **Launch** to preview.

> [!CAUTION]
> If you re-upload a new version of `polygon-streaming.umd.js`, press **Parse** again. The Editor caches the attribute schema — new attributes will not appear until you re-parse.

## Playing Animations

Because the UMD build exposes the same script components, animation control uses the exact same events as the NPM path. From any script that has a reference to the streamable model entity:

```javascript
// Embedded / VRM animation. Second arg: name OR index. Third: transition seconds.
streamableModelEntity.fire('streamable-model:play-animation', 'idle', 0.2);

// VRM expression
streamableModelEntity.fire('vrm-expression:start-emotion', 'happy', {
  times:  [0, 0.1, 0.2],
  values: [0, 1,   0]
});
```

For VRM `.vrma` animations you must set the `vrmAnimations` attribute at create time with `pc.Asset` container assets — same shape as the NPM package. In the Editor you would set this attribute via the inspector (list of `{ name, asset, loop, default }`).

## Attributes

The attribute set is identical to the NPM package. See [`playcanvas-polygon-streaming-npm-package`](../playcanvas-polygon-streaming-npm-package/SKILL.md#stream-controller-attributes-reference) for the full reference tables of `streamController` and `streamableModel` attributes.

Highlights:

- `streamController`: `camera` (required), `cameraType`, `triangleBudget`, `mobileTriangleBudget`, `distanceFactor`, `maximumQuality`, `iOSMemoryLimit`, `occlusionCulling`.
- `streamableModel`: `path` (required), `qualityPriority`, `initialTrianglePercent`, `castShadows`, `receiveShadows`, `useAlpha`, `useEmbeddedCollider`, `animation`, `vrmAnimations`, `environmentAsset`.

## Files Required at the Web Root

Same as the NPM path: the service worker must be at `/service-worker.js`. When you use the PlayCanvas Editor's Launch button, PlayCanvas hosts the launch page — the service worker will not be present, so:

- In the Editor's Launch preview, chunk caching is effectively disabled. Models still load, just not cached across reloads.
- For a **published** PlayCanvas build, add `service-worker.js` to your published site's root. Grab it from the same CDN path as the UMD (`.../<VERSION>/playcanvas/service-worker.js`) or from `packages/core/dist` in the repo.

## Query Parameters

The UMD build honors the same URL params as the NPM path (`md`, `df`, `mq`, `oc`, `webgpu`, `model-only`, `minimize-cpu`, `tlp`, `tls`, etc.). Only available in the internal staging or production player:

- `v` / `version`: pick a specific published player version.
- `pc`: pick a specific PlayCanvas engine version (the preview loads it dynamically).

## Verification Checklist

- [ ] PlayCanvas `<script>` tag comes **before** `polygon-streaming.umd.js`.
- [ ] After the UMD is loaded, `pc.script.create('streamController', ...)` does **not** throw "script not found".
- [ ] `streamableModel` entity is a **child** of the `streamController` entity in the hierarchy.
- [ ] In the PlayCanvas Editor: **Parse** was pressed after upload, and script attributes appear in the inspector.
- [ ] For a published site: `service-worker.js` is served from `/`.

## Gotchas

- **Order matters:** PlayCanvas must exist as the global `pc` before the UMD file executes; otherwise the components never register.
- **Editor caching after re-upload:** press **Parse** again after every new upload of `polygon-streaming.umd.js`. Attribute changes will not show up otherwise.
- **`streamableModel` must be a child of `streamController`.** This is enforced at runtime — a top-level streamable model is silently ignored.
- **Editor Launch has no service worker.** Cold-cache loads on every Launch are expected; don't diagnose it as a perf regression.
- **PlayCanvas version drift.** The standalone works against `playcanvas` 1.x and 2.x, but the shader-chunk APIs it uses assume 2.6+ paths. If your project pins PlayCanvas < 2.6, prefer a matching older release of `polygon-streaming.umd.js`.
- **Don't mix standalone + NPM in one page.** Two copies of the same script components will conflict. Pick one path.
- **Don't rename the uploaded script asset in the Editor.** The internal script names are `streamController` and `streamableModel` regardless of the file name, but keeping the file name conventional avoids confusion.
- **Downloadable-dev is non-minified — do not ship it to production.** It's larger and skips obfuscation.

## References

- Repo build script: `scripts/build.ps1 downloadable` / `downloadable-dev`
- Local UMD preview: `http://localhost:8080/playcanvas/umd/` (after `pnpm run build-dev` inside `packages/playcanvas`)
- Public docs: <https://docs.viverse.com/polygon-streaming/polygon-streaming-javascript-sdk>
- Sister skill (NPM/ESM path): [`playcanvas-polygon-streaming-npm-package`](../playcanvas-polygon-streaming-npm-package/SKILL.md)
