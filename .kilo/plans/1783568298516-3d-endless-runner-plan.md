# 3D Endless Runner — Web Mobile Game

## Goal
Build a fully playable, mobile-first **3D endless runner** web game that runs in any modern
mobile browser, using only **open-source (CC0 / permissive) assets** so there are no licensing
concerns. The result must be a complete, runnable product (not a prototype): full game loop,
touch controls, audio, scoring, and a working production build.

## Confirmed Decisions
- **Stack:** Three.js + TypeScript + Vite (lightweight, huge OSS ecosystem, easy mobile deploy).
- **Gameplay:** 3-lane runner (Subway Surfers style) — swipe L/R to switch lanes, swipe up / tap
  to jump, swipe down to slide; dodge obstacles, collect coins.
- **Art assets:** CC0 low-poly model packs + CC0 audio. Primitive-geometry fallback if a download
  is unavailable, so the game always renders.
- **Scope:** Full mobile game (see Feature Set).
- **Orientation:** Portrait (vertical), one-handed swipe play.

## Open-Source Asset Sources (CC0 / permissive)
Download into `public/assets/` and credit per each license. Prefer glTF/GLB where possible.

**3D Models (CC0):**
- Kenney.nl — https://kenney.nl/assets (e.g. "Low Poly Pack", "City Kit: Streets",
  "Nature Pack", "Characters", "Vehicle Kit"). License: CC0.
- Quaternius — https://quaternius.com (low-poly characters, vehicles, props, environments). License: CC0 / CC-BY (check per pack).
- Poly Pizza — https://poly.pizza (searchable CC0 glTF models, ideal for coins/obstacles/props).
- Sketchfab — https://sketchfab.com (use filters: "Downloadable" + license CC0 / CC-BY). Verify license per model.

**Audio (CC0 / permissive):**
- Kenney Audio — https://kenney.nl/assets (e.g. "Audio Urban", "Audio City"): music loops + SFX, CC0.
- freesound.org — https://freesound.org (filter license: CC0). Jump, coin, crash, UI.
- OpenGameArt — https://opengameart.org (filter license: CC0 / public domain).

**Fallback rule:** If a specific model cannot be fetched, generate an equivalent low-poly mesh
with Three.js primitives (boxes/cylinders/cones) so the build never breaks.

## Feature Set
- Infinite track via recycling road segments + side scenery (object pooling, no GC churn).
- 3 lanes; smooth lane-switch tween; jump (gravity arc) + slide (scale/crouch) with timers.
- Obstacles (barriers, low gates, oncoming objects) spawned with increasing density/speed.
- Collectible coins (score), magnet/value optional stretch.
- Collision detection via AABB; hit → game over.
- Difficulty ramp: forward speed increases with distance/time.
- Game states: `Loading → Menu → Playing → Paused → GameOver → (Restart)`.
- HUD (DOM overlay): score, coin count, high score, pause button, mute button.
- localStorage high score persistence.
- Audio: background music loop + SFX (jump, coin, crash, UI), mute toggle.
- Input: touch swipe + tap (mobile) AND keyboard arrows/space (desktop testing).
- Responsive canvas: resize handling, `devicePixelRatio` capped for perf, portrait layout.
- Loading screen with progress while models/audio load via `LoadingManager`.

## Project Structure
```
index.html
package.json
tsconfig.json
vite.config.ts
public/
  assets/models/   (downloaded GLB/glTF)
  assets/audio/    (downloaded ogg/mp3)
src/
  main.ts                 (bootstrap, render loop, resize)
  game/
    Game.ts               (orchestrator: states, loop, scene, camera)
    Player.ts             (lane logic, jump/slide, animations)
    Track.ts              (recycling road + scenery segments)
    ObstacleManager.ts    (spawn, pool, recycle)
    CoinManager.ts        (spawn, collect, recycle)
    Input.ts              (touch swipe + keyboard)
    Audio.ts              (music + SFX, mute)
    Difficulty.ts         (speed/density ramp)
    Collision.ts          (AABB helpers)
  ui/
    HUD.ts                (DOM overlay: score/coins/pause/mute)
    Screens.ts            (menu / pause / gameover overlays)
  loaders/
    AssetLoader.ts        (GLTF + audio via THREE.LoadingManager)
  config.ts               (tunables: lane width, speeds, spawn rates)
  styles.css
```

## Implementation Steps (ordered)
1. **Scaffold:** `npm create vite` (vanilla-ts) equivalent — write `package.json`,
   `tsconfig.json`, `vite.config.ts`, `index.html`, `src/styles.css`. Add `three` + types.
2. **Bootstrap scene:** renderer (DPR cap, portrait), camera (chase view), lights,
   fog, resize handler, fixed-timestep loop in `main.ts`/`Game.ts`.
3. **Track:** build reusable road segment (3 lanes) + side scenery; recycle segments
   behind the camera to fake infinite distance.
4. **Player + Input:** player mesh on a lane; implement lane switch, jump, slide;
   wire `Input.ts` (swipe/tap + keyboard).
5. **Obstacles & Coins:** `ObstacleManager`/`CoinManager` with pooled spawns keyed to
   difficulty; AABB collision → game over / coin score.
6. **Difficulty ramp:** speed + spawn density scale with distance.
7. **States & UI:** `Loading → Menu → Playing → Paused → GameOver`; DOM HUD + screens;
   localStorage high score; pause/mute buttons.
8. **Audio:** load CC0 music + SFX; play on events; respect mute + autoplay policies
   (start audio after first user gesture).
9. **Assets pass:** download CC0 models/audio into `public/assets`; swap primitives for
   real models; keep primitive fallback. Add credits/license notes file.
10. **Polish & perf:** cap DPR, frustum/render optimizations, simple coin/particle flair,
    mobile-safe touch (prevent scroll/zoom), viewport meta + `touch-action: none`.

## Validation
- `npm run dev` → open on a phone or desktop devtools device emulation (portrait); verify
  full loop: menu → play → swipe lanes/jump/slide → coin scoring → collision → game over →
  restart; high score persists across reloads; audio + mute work.
- `npm run build && npm run preview` → confirm production bundle loads and runs on mobile.
- Sanity: no console errors; stable ~60fps on a mid-range phone; touch does not scroll page.

## Risks / Notes
- External asset downloads require network access during the asset step; primitive fallback
  guarantees a runnable build regardless.
- Verify each downloaded asset's license (Kenney = CC0; Quaternius/Sketchfab vary) and keep
  a `public/assets/CREDITS.md`.
- Audio autoplay: start AudioContext on first user interaction (menu tap).

## Open Questions (non-blocking)
- Theme (city street / forest / sci-fi) — default to **city street** low-poly unless told.
- Deployment host (GitHub Pages / Netlify / static) — build is host-agnostic; pick later.
