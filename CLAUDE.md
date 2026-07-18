# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A clone of the classic arcade game Asteroids, built with pure HTML5 Canvas and vanilla JavaScript (ES6+). No frameworks, no bundler, no dependencies, no package.json.

## Running

Open `index.html` directly in a browser, or serve locally:

```bash
npx serve .
```

There is no build step, linter, or test suite — the entire game is `game.js`, loaded directly by `index.html`.

## Architecture

Everything lives in `game.js` as a single file, organized into clearly delimited sections (marked by `── Section ──` comments) in this order: Input → Utils → Bullet → Asteroid → Ship → Particle → Game state → Update → Draw → Main loop.

- **Coordinate space**: fixed 800×600 (`W`, `H`). All entities wrap toroidally at the edges via the `wrap()` util — there are no walls.
- **Entity classes** (`Bullet`, `Asteroid`, `Ship`, `Particle`) each follow the same shape: constructor sets initial physics state, `update(dt)` advances it, `draw()` renders it to the module-level `ctx`, and a `dead` flag marks it for removal. There is no shared base class or entity manager — the game loop owns per-type arrays (`bullets`, `asteroids`, `particles`) and filters out `dead` entries each frame.
- **Asteroids** have 3 sizes (`size` 1–3, large→small) with per-size radius/speed/points tables (`RADII`, `SPEEDS`, `POINTS`). `Asteroid.split()` produces two smaller asteroids on death; size-1 asteroids don't split.
- **Game state machine**: a single module-level `state` string — `'playing' | 'dead' | 'gameover'` — gates behavior in `update()`. `'dead'` is a temporary post-collision state (`deadTimer`, ship blinks/respawns with temporary invincibility); `'gameover'` waits for Space to call `initGame()` again.
- **Input**: raw key state lives in `keys` (held) and `justPressed` (edge-triggered, consumed via `pressed(code)`) — set by `keydown`/`keyup` listeners at the top of the file. `ArrowLeft`/`ArrowRight` rotate, `ArrowUp` thrusts, `Space` shoots/restarts.
- **Main loop**: `requestAnimationFrame`-driven `loop(ts)` computes `dt` (clamped to 0.05s to avoid physics blowups on tab-switch), then calls `update(dt)` followed by `draw()`. `initGame()` runs once at load to bootstrap state.
- **Collision detection** is simple circle-distance checks (`dist()` + summed radii), done directly inside `update()` for bullet-vs-asteroid and ship-vs-asteroid — no spatial partitioning, fine at this entity count.

When adding new entity types or mechanics, follow the existing per-type-array + `dead`-flag-filtering pattern rather than introducing a generic entity manager.
