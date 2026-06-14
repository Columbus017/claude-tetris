# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step. Open directly or serve with any static server:

```bash
open index.html                  # macOS
python3 -m http.server 8000      # then open http://localhost:8000
npx serve .
```

## Architecture

Three files, no dependencies:

- **`index.html`** — DOM structure: `<canvas id="board">` (300×600 px) for the game field, `<canvas id="next-canvas">` (120×120 px) for the next-piece preview, HUD spans (`#score`, `#lines`, `#level`), and `#overlay` for pause/game-over states.
- **`style.css`** — dark/retro theme; overlay uses `backdrop-filter`.
- **`game.js`** — all game logic (~300 lines, no modules).

### game.js internals

- **Board**: `ROWS×COLS` matrix; `0` = empty, `1–7` = piece color index.
- **Piece object**: `{ type, shape, x, y }` where `shape` is a 2-D matrix and `(x, y)` is the top-left grid position.
- **Collision**: `collide(shape, ox, oy)` — checks bounds and board occupancy.
- **Rotation**: `rotateCW` (transpose + row-reverse); `tryRotate` applies wall kicks `[0, -1, 1, -2, 2]`.
- **Game loop**: `requestAnimationFrame`-based; `dropAccum` accumulates elapsed ms and triggers a row drop when it exceeds `dropInterval`.
- **Speed formula**: `dropInterval = max(100, 1000 − (level − 1) × 90)` ms.
- **Line clear**: `clearLines` splices full rows bottom-up and unshifts empty rows at the top.
- **Ghost piece**: `ghostY()` projects the current piece downward; drawn at `globalAlpha = 0.2`.
- **Entry point**: `init()` — resets all state and starts the loop. Called on load and on restart button click.

### Key tunable constants in game.js

| Constant | Default | Note |
|---|---|---|
| `COLS` / `ROWS` | 10 / 20 | Also update canvas `width`/`height` in index.html (`COLS×BLOCK`, `ROWS×BLOCK`) |
| `BLOCK` | 30 px | Cell size in pixels |
| `COLORS` | 7 colors | Index 1–7 maps to piece types I/O/T/S/Z/J/L |
| `LINE_SCORES` | `[0,100,300,500,800]` | Multiplied by current level |
