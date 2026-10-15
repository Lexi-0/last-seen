# LAST SEEN

*You heard your friend. Your friend isn't there.*

A first-person survival horror for mobile and desktop browsers, set in a fictional Nigerian university hostel (Ireti Hall, Oyedara University) during a storm. Find out what happened to Tobi, restore the power, and get out through the gate while **the Mimic**, a creature that repeats voices it has heard, hunts you by sound and sight.

Original story and creature. In-game rumours are framed as campus talk; nothing here claims to be traditional belief. Free to play, no account, no payments, no microphone access.

## Status (read this first)

| Area | State |
|---|---|
| Solo Chapter One (full mission loop) | Implemented. Mission logic verified by automated test through the real command API |
| The Mimic AI (6-state FSM) | Implemented. Fairness rules verified by automated tests |
| Rendering, audio, UI, touch controls | Implemented and type-checked. **Not verified visually or audibly beyond the menu and first frame** (see Testing) |
| Mobile performance | **Unmeasured.** Adaptive resolution and a Low preset exist; real-device numbers are still needed |
| Private multiplayer | **Not implemented.** Menu button is disabled. `src/sim/` is written so a Node server can run it |
| Public matchmaking | Not implemented |
| Voice lines | Text-to-speech + formant "mumble" placeholders, not recorded VO. Subtitles always show |

## Install / run

Requires Node 20+.

```bash
npm install
npm run dev        # http://localhost:5173 (also on your LAN for phone testing)
npm run build      # typecheck + production bundle in dist/
npm run preview    # serve dist/ on :4173
npm test           # vitest: level, mission, Mimic AI and stats-server tests
npm start          # serve dist/ + stats API (after npm run build)
```

Serve `dist/` from any static host. Open with `?debug` to expose `window.__ls` (test hooks). Enable *Show FPS / debug* in Settings for fps, draw calls and triangles.

## Play stats (how many people opened the link and played)

`server/` is a small zero-dependency Node server that serves the built game **and** counts anonymous events. Static-only hosts (Netlify Drop and similar) cannot count anything, so to get numbers the game must be served by this server, or built with `VITE_STATS_URL=https://your-stats-host` so it reports to a server running elsewhere.

```bash
npm run build
STATS_TOKEN=choose-a-password npm start     # game on :8080, stats at http://localhost:8080/stats
```

- **Counted:** link opens (page loads, once per tab session), unique visitors, runs started (Play pressed), unique players, escapes, deaths, average escape time, average evidence, mobile vs desktop, last 14 days, and link sources. Share links like `https://your-host/?src=whatsapp` to see which channel brought people.
- **Private:** the `/stats` page and `/api/stats` need the `STATS_TOKEN` password. Without it the server prints a temporary token at start. Set it in your host's environment settings.
- **Privacy:** no IP addresses or personal data are stored. A random anonymous ID lives in the player's browser. Players can switch it off in Settings ("Anonymous play stats"), and Do Not Track is respected. Mention this in your store listing or page if you publish widely.
- **Persistence:** data is a JSON file in `DATA_DIR` (default `./data`). On hosts with a temporary disk it resets on redeploy; attach a persistent disk or move to a hosted database before relying on the numbers.
- **Limits:** it counts page loads and button presses, not people. Clearing site data creates a new ID, and ad blockers or Do Not Track hide some users. Event posts are rate-limited per IP in memory.
- **Run on any host that runs Node** (build command `npm install && npm run build`, start command `npm start`, set `STATS_TOKEN`).

## Controls

**Desktop:** WASD move · mouse look (click to capture) · Shift run · C crouch · F flashlight · E use (hold E at the gate) · Esc or P pause.

**Mobile (landscape):** left stick move · drag right side to look · USE · RUN (hold) · DUCK (toggle) · LIGHT · pause button. Sensitivity, brightness, volume, graphics preset, invert-Y, voice and item glints are in Settings.

## How the Mimic works (so it is learnable)

- Dormant at start; wakes when you enter the dorm wing (or after about 70 s).
- **Hearing:** about 15 m sprinting, 7 m walking, 2.5 m crouched, 0 when still or hiding. Doors, pickups, the generator and the gate make noise. Walls muffle it.
- **Sight:** line of sight plus its forward cone. About 16 m with your torch on, 7 m in the dark.
- **Warnings:** nearby lights stutter, your torch flickers, static and heartbeat rise, it screeches before a chase (0.8-1.5 s) and growls before a strike (0.65 s, ending in a short pounce).
- **Mercy:** 2 hits to die. After a hit you are invulnerable for 3.6 s and it withdraws for 10 s. It gives up after about 6 s without sight, or 38 s of chasing.
- **Hiding:** it only checks a wardrobe or locker if it saw you go in.
- **Voice lures:** it repeats fixed phrases exactly the same way each time. That is the tell.
- No teleporting in normal play. The only relocation is at the finale start (and a stuck rescue), always out of sight and at least 26 m away by path.

## Layout

```
src/sim/      pure TypeScript, no DOM/Three.js (server-ready)
  level.ts      grid, rooms, doors, props, items, hiding spots, lamps
  collision.ts  circle-vs-AABB, segment tests, 0.5 m nav grid + A*
  mimic.ts      the Mimic state machine and tuning constants
  sim.ts        world state, commands, doors, generator, gate, scares, objectives
src/client/   Three.js renderer, procedural textures/geometry, input, WebAudio, UI
tests/        vitest suites
```

## Testing

Done: `tsc --noEmit`, production build, 17 unit/integration tests (enclosure and reachability, locked-door gating, full scripted mission to a win, Mimic telegraph / wind-up / mercy rules, speed cap and no-teleport over 10 simulated minutes, finale relocation rules). Browser smoke test: the menu and first in-game frame render in headless Chromium (software GL) with no console errors.

Not done: visual review of all areas, touch testing, audio listening, FPS on real Android devices. In the headless sandbox used for development, reading pixels back from the game's frame crashed the GPU process while a trivial WebGL canvas did not. That may be a SwiftShader limitation or a real rendering bug, and **must be checked on a real GPU first**.

## Roadmap

1. Real-device pass (Android Chrome, desktop): fix visuals, lighting levels and UI fit; measure FPS; cut draw calls (door handles are separate meshes).
2. Private multiplayer: `ws` server running `Sim` at 20 Hz, 4-letter room codes, clients send poses/commands, server owns doors, items, generator, gate, objectives, Mimic and hits, with movement validation (`Sim.opts.validateMovement` exists). Verify with two real clients.
3. Public matchmaking on top of the room-code layer.
4. Recorded voice lines, baked lightmaps, more chapters.
