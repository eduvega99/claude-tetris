# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A single-page Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package.json, no test suite.

## Running the game

There is no build/lint/test tooling. To run:

```bash
xdg-open index.html          # open directly, or
python3 -m http.server 8000  # then visit http://localhost:8000
```

To verify a change works, open `index.html` in a browser and play through the affected mechanic manually (there is no automated test suite).

## Architecture

Three files, all loaded by `index.html`:

- `index.html` — DOM structure: the `#board` canvas (300×600), a side panel (score/lines/level/next-piece canvas/controls), and the pause/game-over `#overlay`.
- `style.css` — dark/retro arcade visual theme.
- `game.js` — all game logic, structured around a small set of global `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `animId`) mutated by top-level functions. No modules, no classes.

### Core model

- **Board**: `ROWS × COLS` matrix (`createBoard`), each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
- **Pieces**: the 7 tetrominoes are defined as square matrices in `PIECES` (index 0 unused/null so piece type doubles as array index and color index into `COLORS`). Rotation is done by transposing + reversing rows (`rotateCW`), not by pre-baked rotation states.
- **Collision** (`collide`): checks board bounds and overlap with locked cells; used for movement, rotation, and ghost-piece projection.
- **Wall kicks** (`tryRotate`): after rotating, tries horizontal offsets `[0, -1, 1, -2, 2]` until a non-colliding position is found, else the rotation is discarded.

### Game loop

`loop(ts)` runs via `requestAnimationFrame`, accumulating elapsed time in `dropAccum` and advancing the piece one row once `dropAccum >= dropInterval`, otherwise locking it (`lockPiece` → `merge` + `clearLines` + `spawn`).

- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` × current `level`; hard drop adds 2 pts/row dropped, soft drop adds 1 pt/row.
- **Leveling/speed**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece**: `ghostY()` projects the current piece straight down to its landing row; drawn at `globalAlpha = 0.2`.
- **Game over**: triggered in `spawn()` when a freshly spawned piece immediately collides.

Input is handled by a single `keydown` listener mapping arrow keys / `X` / `Space` / `P` to movement, rotation, soft/hard drop, and pause.

## Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK` (cell px size), `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, also update the `width`/`height` attributes of `<canvas id="board">` in `index.html` to match (`COLS × BLOCK` and `ROWS × BLOCK`).
