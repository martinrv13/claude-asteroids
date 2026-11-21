# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

Open `index.html` directly in a browser, or use a local server:

```bash
npx serve .
```

Then visit `http://localhost:3000`. There is no build step, no package.json, and no dependencies.

## Architecture

The entire game lives in a single file: `game.js`. The canvas is 800×600 (`W`/`H` constants), accessed via a global `ctx`.

**Entity classes** — each has `update(dt)` and `draw()` methods:
- `Bullet` — fired from the ship nose, killed by TTL or asteroid hit
- `Asteroid` — size 1/2/3 (small/medium/large); `split()` returns two size-1 children; polygon shape is randomized at construction
- `Ship` — handles thrust/rotation/shooting; `invincible` timer prevents collisions after respawn; `dead` flag suspends updates/drawing
- `Particle` — short-lived explosion streak, fades by alpha
- `PowerUp` — 5-way shot pickup. The ship always fires a 3-bullet fan (spread `TRIPLE_SPREAD`). The pickup drops at a destroyed asteroid's position with `TRIPLE_DROP_CHANCE`, at most once per level (`tripleSpawned`, reset in `initGame()` and `nextLevel()`). Expires after `POWERUP_TTL`. Collecting it sets `ship.tripleShot = TRIPLE_DURATION` (10 s); `Ship.tryShoot()` then fires a 5-bullet fan. Effect is lost on death, kept across levels.

**Game loop** — `requestAnimationFrame`-based `loop(ts)`. Delta time `dt` is capped at 50 ms (0.05 s) to prevent tunneling after tab-switch.

**State machine** — `state` ∈ `{ 'playing', 'dead', 'gameover' }`. The `update()` function branches on this; `'dead'` runs a 2-second `deadTimer` before respawning.

**Input** — `keys{}` holds currently-held keys; `justPressed{}` captures single-frame presses. `pressed(code)` reads and clears a just-pressed entry — call it once per frame per key or the second read returns false.

**Asteroid size constants** are arrays indexed by size (index 0 unused):
- `RADII[size]`, `SPEEDS[size]`, `POINTS[size]`

**Collision detection** uses circle overlap (`dist(a, b) < a.radius + b.radius`). Ship collision applies a 0.82 multiplier on the asteroid radius for a slightly forgiving hitbox.

**Level progression** — when `asteroids.length === 0`, `nextLevel()` is called, which increments `level` and spawns `3 + level` new large asteroids.
