# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A Tetris clone in vanilla JavaScript + HTML5 Canvas. No dependencies, no `package.json`, no bundler, no transpiler, no tests, no linter. README, UI strings (`PAUSA`, `Reiniciar`, `Puntuación`…) and `<html lang="es">` are in Spanish — keep new user-facing text in Spanish.

## Running

Open `index.html` directly (`start index.html` on Windows), or serve the folder statically and visit `http://localhost:8000`:

```bash
python -m http.server 8000
# or: npx serve .
```

There is no build step; reload the browser to see changes.

## Architecture

Three files: `index.html` (DOM + two canvases + overlay), `style.css` (dark theme), `game.js` (all logic, loaded as a classic `<script>` at the end of `<body>`, not a module).

Key points in `game.js` that span several functions:

- **Piece type = color index = cell value.** `PIECES[i]` and `COLORS[i]` are parallel arrays indexed 1–7 (index 0 is `null`). Each shape matrix stores its type index in its filled cells, so `merge()` copies those values straight into `board`, and `drawBlock()` looks up the color from the cell value. Adding or reordering pieces requires keeping both arrays aligned, and `randomPiece()` hardcodes `* 7`.
- **All game state is module-level `let` globals** (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `dropAccum`, `lastTime`, `animId`), reset in `init()`. The restart button calls `init()` directly.
- **Game loop:** `loop()` runs on `requestAnimationFrame`, accumulates `dt` into `dropAccum`, and drops the piece one row (or calls `lockPiece()`) when it exceeds `dropInterval`. It reschedules itself unconditionally at the end, so pausing/ending relies on `cancelAnimationFrame(animId)`.
- **Locking pipeline:** `lockPiece()` → `merge()` → `clearLines()` (updates lines/score/level/`dropInterval`) → `spawn()` (promotes `next` to `current`; if the new piece collides immediately, `endGame()`).
- **Collision:** `collide(shape, x, y)` treats cells with `y < 0` as free (pieces spawn at `y = 0`; there are no hidden rows above the board). Rotation (`tryRotate`) uses `rotateCW` plus simple horizontal kicks `[0, -1, 1, -2, 2]` — not SRS.
- **Rendering:** `draw()` redraws grid → locked board → ghost piece (`ghostY()`, alpha 0.2) → current piece every frame. The next-piece preview (`drawNext()`) is only redrawn on `spawn()`. HUD text is updated via `updateHUD()`, called from the keydown handler and from scoring paths.
- **Canvas size is coupled to constants:** `<canvas id="board">` in `index.html` must be `COLS × BLOCK` by `ROWS × BLOCK` (300×600). The preview canvas is 120×120 and `drawNext()` assumes a 4×4 grid of 30px cells.
