# web-polygon-streaming-skills

Skills bundle for **standalone (non-VIVERSE-published)** Polygon Streaming Web Player integrations (`web-polygon-streaming` repo).

Polygon Streaming streams high-poly `.xrg` models to a web page and progressively refines geometry / textures based on a triangle budget and camera distance.

> [!IMPORTANT]
> **If your PlayCanvas project will be published to VIVERSE**, use the [`viverse-playcanvas-toolkit`](https://github.com/viverseofficial/viverse-playcanvas-toolkit) bundle instead — specifically the [`viverse-polygon-streaming`](https://github.com/EJHuang-HTC/playcanvas-toolkit-skills/tree/main/skills/viverse-polygon-streaming) skill. That path uses the VIVERSE PlayCanvas Toolkit Chrome extension and does not require any code.
>
> This bundle covers the **other integration paths**: bundler-based NPM (JavaScript SDK), Standalone Plugin (Option 2), HTML Scripting (Option 3), Three.js, and Babylon.js.

## Skills in this bundle

### Polygon Streaming PlayCanvas SDK — non-VIVERSE paths

| Skill | Official Doc Section | When to use |
| --- | --- | --- |
| [`playcanvas-polygon-streaming-standalone-plugin`](skills/playcanvas-polygon-streaming-standalone-plugin/SKILL.md) | **Option 2: Standalone Plugin** | Project is **not** published to VIVERSE. Upload `polygon-streaming.js` into the PlayCanvas Editor as a script asset. No bundler. |
| [`playcanvas-polygon-streaming-html-scripting`](skills/playcanvas-polygon-streaming-html-scripting/SKILL.md) | **Option 3: HTML Scripting** | Project is **not** published to VIVERSE. Edit a downloaded HTML build and pull `PolygonStreaming.js` from the Viverse CDN. |

### Polygon Streaming JavaScript SDK

For custom apps using NPM + a bundler (Vite/Webpack), following the official *JavaScript SDK* structure:

| Skill | Official Doc Section | When to use |
| --- | --- | --- |
| [`playcanvas-polygon-streaming-npm-package`](skills/playcanvas-polygon-streaming-npm-package/SKILL.md) | JavaScript SDK → **PlayCanvas** | Bundler-based PlayCanvas app importing `@polygon-streaming/web-player-playcanvas`. |
| [`threejs-polygon-streaming`](skills/threejs-polygon-streaming/SKILL.md) | JavaScript SDK → **Three.js** | Bundler-based Three.js app importing `@polygon-streaming/web-player-threejs`. |

### Unofficial / repo-only

| Skill | Status | When to use |
| --- | --- | --- |
| [`babylonjs-polygon-streaming`](skills/babylonjs-polygon-streaming/SKILL.md) | ⚠️ **Not in official Viverse docs** | Only if a Babylon.js integration is required. Prefer Three.js or PlayCanvas when possible. |

## Choosing the right skill

```
Are you publishing to VIVERSE?
├── YES  → USE THE OTHER BUNDLE:
│         playcanvas-toolkit-skills/
│           ├── viverse-toolkit-setup (install extension first)
│           └── viverse-polygon-streaming (add PolygonStreaming to entity)
│
└── NO — which engine?
    ├── PlayCanvas + bundler (Vite/Webpack)   → playcanvas-polygon-streaming-npm-package
    ├── PlayCanvas Editor, no bundler         → playcanvas-polygon-streaming-standalone-plugin
    ├── PlayCanvas downloaded HTML build      → playcanvas-polygon-streaming-html-scripting
    ├── Three.js                               → threejs-polygon-streaming
    └── Babylon.js (unofficial)               → babylonjs-polygon-streaming
```

## Terminology alignment

- **Stream Controller** — scene-level component / class.
- **Streamable Model** — per-model component / `addModel` method.
- **Model URL / `path`** — the Asset ID URL from stream.viverse.com/console.
- **`.xrg`** — the streamable model file format.
- **Attribute aliases**: `iOSMemoryLimit` ≡ `iosMemoryLimit`, `forceDoubleSided` ≡ `doubleSidedMaterials`.

## Related bundles

- **`playcanvas-toolkit-skills`** — VIVERSE PlayCanvas Toolkit (Chrome extension + Framework + PolygonStreaming module).
- **`viverse-sdk-skills`** — reference bundle for VIVERSE Web SDK.

## Official Viverse Docs

- Polygon Streaming JavaScript SDK: <https://docs.viverse.com/polygon-streaming/polygon-streaming-javascript-sdk>
- Polygon Streaming PlayCanvas SDK: <https://docs.viverse.com/polygon-streaming/integrations-and-plugins/polygon-streaming-playcanvas-sdk>
- Model console: <https://stream.viverse.com/console>
