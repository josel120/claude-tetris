# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla-JS Tetris: three files (`index.html`, `style.css`, `game.js`), no dependencies, no `package.json`, no build step, no test suite.

## Running

Open `index.html` directly, or serve statically (`python3 -m http.server 8000` / `npx serve .`). Verification is manual in a browser — there is no lint, build, or test command to run.

## Architecture (`game.js`)

Single classic script (`<script src="game.js">`, no modules, no bundler). Everything lives in top-level scope; DOM elements are grabbed once into consts at load, and mutable game state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `dropAccum`, `animId`) sits in module-level `let`s.

Key design points that aren't obvious from a single file read:

- **Cell values are piece type *and* color index at once.** Every `PIECES[n]` matrix is filled with the literal `n` (the T piece is made of `3`s), and that same number indexes `COLORS`. `board` cells store that number; `0` means empty. So a shape matrix is never boolean — changing a piece's numbers changes its color, and `COLORS[0]` is deliberately `null`.
- **`init()` is both boot and restart.** It's called at the bottom of the file, and by the restart button. It cancels any in-flight `animId` before starting a new `requestAnimationFrame` loop — keep that guard when adding restart paths, or you get two loops running at double speed.
- **Pause works by cancelling the rAF loop**, not by a flag inside `loop()`. On resume `togglePause()` resets `lastTime = performance.now()` before restarting; skipping that reset makes `dt` equal the whole paused duration and instantly drops the piece.
- **Rotation** is transpose-and-reverse (`rotateCW`) plus a fixed kick table `[0, -1, 1, -2, 2]` in `tryRotate` — not SRS. If a kick offset collides, the rotation is silently discarded.
- **`clearLines`** splices the full row out and unshifts an empty one at the top, then does `r++` to re-examine the same index (which now holds the row that shifted down). Preserve that when touching the loop.
- **`drawBlock`** temporarily mutates `context.globalAlpha` and resets it to `1` on exit; the ghost piece is just the same function called with `0.2`. It's shared by the board canvas and the next-piece canvas, hence the explicit `context` and `size` params.
- **`draw()` clears and repaints the whole canvas every frame** (grid → locked board → ghost → current piece, in that order). There is no dirty-rect or layer caching.

## Constraint when changing board dimensions

`COLS`, `ROWS`, and `BLOCK` in `game.js` have no link to the canvas element. If you change any of them, manually update `width`/`height` on `<canvas id="board">` in `index.html` to `COLS * BLOCK` × `ROWS * BLOCK`. The next-piece preview uses its own hardcoded `NB = 30` in `drawNext()` against a 120×120 canvas and assumes a 4×4 centering grid.

## Language

User-facing strings (overlay text, README) are in Spanish; code identifiers and comments are English/Spanish mixed. Match the surrounding file when adding text.
