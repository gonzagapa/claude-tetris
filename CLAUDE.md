# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No package.json, no build, no lint, no tests. UI text and README are in Spanish.

## Run

Open `index.html` directly, or serve statically: `python -m http.server 8000` then http://localhost:8000.

## Architecture

Three files: `index.html` (DOM + canvases), `style.css`, `game.js` (all logic, ~300 lines, global scope, `'use strict'`).

- `index.html` ids are looked up by `game.js` at load (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`). Renaming one breaks the script.
- Canvas `#board` is hardcoded `300x600` = `COLS*BLOCK` x `ROWS*BLOCK`. Changing `COLS`/`ROWS`/`BLOCK` in `game.js` requires updating the canvas `width`/`height` in `index.html`.
- Board is `ROWS x COLS` matrix; cell is `0` or color/piece index 1-7. `PIECES` and `COLORS` share the same index (1=I, 2=O, 3=T, 4=S, 5=Z, 6=J, 7=L); shape cells hold their own index.
- State is module-level `let` vars (`board, current, next, score, ...`), reset by `init()` (also the restart button handler).
- Game loop: `loop(ts)` via `requestAnimationFrame`, accumulates `dropAccum` against `dropInterval`. `animId` is cancelled on pause/game over; resume (`togglePause`) calls `loop` directly after resetting `lastTime`.
- Piece lifecycle: `lockPiece()` -> `merge()` -> `clearLines()` (updates score/level/`dropInterval`) -> `spawn()`. `spawn()` calls `endGame()` if the new piece collides.
- Rotation: `rotateCW` + `tryRotate` with horizontal-only kicks `[0,-1,1,-2,2]` (no SRS, no floor kicks).
- Scoring: `LINE_SCORES[cleared] * level`; soft drop +1/row, hard drop +2/row. Level = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`.
- `ghostY()` is reused by `hardDrop()` and `draw()`.
- Keyboard handler uses `e.code` (layout-independent) and calls `updateHUD()` after every keypress.
