# web-polygon-streaming-skills

Skills bundle for the Viverse [Polygon Streaming](https://stream.viverse.com) Web Player (`web-polygon-streaming` repo).

Polygon Streaming streams high-poly `.xrg` models to a web page and progressively refines geometry/textures based on a triangle budget and camera distance. It ships as three separate 3D-engine bindings on top of a shared `core` package.

## Skills in this bundle

| Skill | Engine | When to use |
| --- | --- | --- |
| [`playcanvas-polygon-streaming-npm-package`](skills/playcanvas-polygon-streaming-npm-package/SKILL.md) | PlayCanvas | You have a bundler (Vite/Webpack) and `import` from `@polygon-streaming/web-player-playcanvas`. Used by Viverse Create, editor extension, custom PlayCanvas apps. |
| [`playcanvas-polygon-streaming-standalone`](skills/playcanvas-polygon-streaming-standalone/SKILL.md) | PlayCanvas | You want a single UMD `polygon-streaming.umd.js` file to drop into the PlayCanvas Editor or a plain `<script>` tag. No bundler. The repo README calls this the "Downloadable Version". |
| [`threejs-polygon-streaming`](skills/threejs-polygon-streaming/SKILL.md) | Three.js | You are integrating into a Three.js scene via `@polygon-streaming/web-player-threejs`. |
| [`babylonjs-polygon-streaming`](skills/babylonjs-polygon-streaming/SKILL.md) | Babylon.js | You are integrating into a Babylon.js scene via `@polygon-streaming/web-player-babylonjs`. |

## Structure

Each skill follows the standard skill layout:

```
skills/<skill-name>/
  skill.json    # metadata (id, tags, prerequisites, entrypoint)
  SKILL.md      # the actual instructions
```

## Repo reference

Source repo: `web-polygon-streaming`

- `packages/core` — engine-agnostic streaming logic
- `packages/playcanvas` — PlayCanvas binding (`@polygon-streaming/web-player-playcanvas`)
- `packages/threejs` — Three.js binding (`@polygon-streaming/web-player-threejs`)
- `packages/babylonjs` — Babylon.js binding (`@polygon-streaming/web-player-babylonjs`)
- Preview app: `src/{playcanvas,threejs,babylonjs}` — runnable via `pnpm run dev`
