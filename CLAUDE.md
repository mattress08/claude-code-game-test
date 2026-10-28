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

- `state` is `'title' | 'play' | 'paused' | 'over'`. The title screen exists so the first click can start audio; ESC toggles pause (and the music).
- All game state is module-level globals, (re)initialised by `reset()` (also generates the random walls, keeping the player spawn clear); `start()` = `reset()` + intro sound + music.
- `loop()` runs on `requestAnimationFrame`, caps `dt` at 0.05 s, calls `update(dt)` only in `play`, `updateFx(dt)` (particles, shake, flash, banners) in `play` and `over`, then `draw()` always.
- Collision is circle-vs-rect via `hitsRect` / `hitsWall`; `move()` resolves X and Y separately so things slide along walls, and returns which axes were blocked (the player uses it to zero momentum).
- Player has momentum: input adds `ACCEL`, `FRICTION` decays velocity, top speed is `ACCEL / FRICTION`. Hits apply knockback through the same velocity.
- Enemies only shoot when `los()` (sampled line of sight) reaches the player, which is what makes walls work as cover.
- Waves: wave 0 counts as cleared, so an `INTERMISSION` countdown precedes every wave; wave `n` spawns `waveSize(n)` enemies and they speed up per wave.
- All player damage goes through `damage()` (shield, HP, invulnerability, death); all enemy deaths go through `kill()`, which grants a random `POWERS` entry every `POWER_EVERY` kills. Nova kills pass `countsForPower = false` so powerups can't chain.
- Impact feel comes from `shake` (world is translated, HUD is not), `flash`/`flashRGB` (red on hit), and `burst()` particles.
- Tuning constants sit at the top of the script; the neon look comes from the `glow()` helper (`shadowBlur`).

## Assets

`sfx/` holds the `.wav` sounds used by `shooter.html` through the `sfx(name, vol)` helper (clones the element so repeats overlap) and the looping `music` element. Because pages run from `file://`, load audio with `new Audio('sfx/<name>.wav')`; `fetch`/Web Audio decoding of local files is blocked in Chrome. Browsers block audio until a user gesture. `background_music.wav` is ~8 MB.
