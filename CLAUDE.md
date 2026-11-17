# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris in vanilla JavaScript + HTML5 Canvas. No dependencies, no `package.json`, no build step, no tests, no linter. The README and UI text are in Spanish; keep new user-facing strings in Spanish.

## Running

Open `index.html` directly, or serve statically (e.g. `python -m http.server 8000`, then http://localhost:8000). There is no test suite; verify changes by playing in the browser.

## Architecture

Three files, all logic in `game.js` (a single script loaded via a classic `<script>` tag, `'use strict'`, global state declared with `let` at the top):

- `index.html` — fixed DOM ids that `game.js` looks up at load: `board` (300×600 canvas), `next-canvas` (120×120), `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`. Renaming any of these breaks the game.
- `style.css` — styling only; the overlay is shown/hidden via the `hidden` class.
- `game.js` — game state and logic:
  - Board is a `ROWS×COLS` matrix of `0` or a piece-type index 1–7; the same index selects the entry in `COLORS` and `PIECES` (index 0 is `null` in both).
  - Pieces are square matrices; rotation is `rotateCW` plus horizontal-only kicks `[0,-1,1,-2,2]` in `tryRotate`. Pieces spawn at `y: 0`, and `collide` tolerates `ny < 0`.
  - Piece lifecycle: `lockPiece()` → `merge()` → `clearLines()` (updates score/level/`dropInterval`) → `spawn()`, which calls `endGame()` if the new piece collides immediately.
  - Loop: `loop(ts)` is a `requestAnimationFrame` loop accumulating `dropAccum` against `dropInterval`. Pause and game over stop it with `cancelAnimationFrame(animId)`; unpausing restarts it manually, and `init()` also serves as restart.
  - Input is a single `keydown` handler (`event.code`-based); gravity, soft drop and hard drop all go through `collide`/`lockPiece`.

## Gotchas

- Changing `COLS`, `ROWS` or `BLOCK` requires updating the `<canvas id="board">` `width`/`height` in `index.html` (`COLS*BLOCK` × `ROWS*BLOCK`).
- The README says the next-piece view uses a 4×4 grid of 30px cells (`drawNext`); keep that in sync with the 120×120 canvas.
- Score/speed formulas live in `clearLines` (`LINE_SCORES[cleared] * level`, `dropInterval = max(100, 1000 - (level-1)*90)`); hard drop awards 2 points per cell and soft drop 1.
