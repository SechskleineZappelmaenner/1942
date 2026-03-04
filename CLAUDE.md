# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-file HTML5 arcade shooter game (`index.html`) — a 1942-style vertical scrolling shooter. No build system, no dependencies, no server required. Open directly in a browser.

## Architecture

Everything lives in `index.html` as one self-contained `<script>` block. Key sections in order:

| Section | What it does |
|---|---|
| **AUDIO** | Web Audio API — procedural sound via `tone()` (oscillator) and `noise()` (white noise buffer). All sounds defined in `SFX` object. |
| **INPUT** | `keys` (held) + `jp` (just-pressed, cleared each frame via `clearJP()`). `jp` is used for one-shot actions like barrel roll. |
| **BACKGROUND** | Scrolling ocean with wave lines and randomly generated islands (`genIslands()`). Islands scroll at 45% of `bgY` for parallax. |
| **PARTICLES** | Simple array of particle objects; `explode()` spawns them, `tickParticles()` / `drawParticles()` update/render. |
| **DRAW: PLANES** | Pure canvas path drawing — `drawPlayer`, `drawFighter`, `drawBomber`, `drawBossPlane`. No image assets. |
| **BULLETS** | `playerBullets` / `enemyBullets` arrays of plain objects `{x, y, vx, vy}`. |
| **Player `P`** | Plain object with `update()` / `draw()` / `hit()` methods. Barrel roll sets `rolling=true` and uses `rollAngle` for `ctx.scale()` distortion. |
| **Enemy class** | Follows optional cubic Bézier `path` (4 control points), then free movement. Three types: `fighter`, `bomber`, `kamikaze`. |
| **Boss class** | Three attack phases based on HP thresholds. Spawns additional fighters periodically. |
| **POWER-UPS** | `double`, `spread`, `shield`, `life`. Applied on player overlap. Timer-based (`shotTimer`) resets shot type to `single`. |
| **WAVE SYSTEM** | `spawnWave()` uses `setTimeout` delays to stagger spawns. `waveSpawned` flag + empty `enemies` array triggers next wave. Every 8th wave is a boss wave. |
| **MAIN LOOP** | `update()` → `render()` via `requestAnimationFrame`. `STATE` controls flow: `menu` / `playing` / `paused` / `gameover`. |

## Key conventions

- Collision uses simple AABB: `hit(ax,ay,aw,ah, bx,by,bw,bh)` — checks overlap of axis-aligned rectangles.
- `curvePath(x1,y1, x2,y2, cx1,cy1, cx2,cy2)` returns a 4-point array for cubic Bézier enemy paths.
- Enemy Bézier evaluation is inline in `Enemy.update()` (de Casteljau, `pathT` 0→1).
- High score persisted via `localStorage` key `1942hi`.
- `frame` counter increments every loop tick regardless of game state (used for blinking UI).
