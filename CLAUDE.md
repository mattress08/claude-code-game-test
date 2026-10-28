# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running

No build step, dependencies, linter or test runner. Each game is one self-contained HTML file (inline CSS + JS) opened straight from disk:

```powershell
Start-Process "shooter.html"
```

The only tests are `console.assert` self-checks at the top of each script; they run on page load and report in the browser console (F12).

## Conventions

- One HTML file per game, no external libraries or CDN scripts.
- Comments are short and only explain the why.
- Known limitations are marked `shortcut: <limit>, <when to upgrade>` so they can be found later.

## shooter.html architecture

- All game state is module-level globals, (re)initialised by `reset()` (also generates the random walls, keeping the player spawn clear).
- `loop()` runs on `requestAnimationFrame`, caps `dt` at 0.05 s, calls `update(dt)` (skipped when `over`) then `draw()`.
- Collision is circle-vs-rect via `hitsRect` / `hitsWall`; `move()` resolves X and Y separately so the player and enemies slide along walls.
- Enemies only shoot when `los()` (sampled line of sight) reaches the player, which is what makes walls work as cover.
- Tuning constants (`PR`, `ER`, `BR`, `PSPEED`, `ESPEED`, `PBSPEED`, `EBSPEED`, `FIRE_RATE`) sit at the top of the script.
- HUD and game-over screen are drawn on the canvas in `draw()`; the neon look comes from the `glow()` helper (`shadowBlur`).

## Assets

`sfx/` holds `.wav` sound effects and music, not yet wired into any game. Because pages run from `file://`, load them with `new Audio('sfx/<name>.wav')` (relative path); `fetch`/Web Audio decoding of local files is blocked in Chrome. Browsers block audio until a user gesture, so start music on the first click/keypress. `background_music.wav` is ~8 MB.
