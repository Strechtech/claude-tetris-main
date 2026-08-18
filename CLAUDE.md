# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Game

No build process. Open `index.html` directly or use a local server:

```bash
# Python 3
python3 -m http.server 8000
# then http://localhost:8000

# Node.js
npx serve .

# PHP
php -S localhost:8000
```

Or simply: `start index.html` (Windows), `open index.html` (macOS), `xdg-open index.html` (Linux).

## Project Structure

- `index.html` — DOM structure, two canvas elements (main board + next-piece preview)
- `style.css` — dark arcade theme, flexbox layout, backdrop blur overlays
- `game.js` — all game logic (~300 lines)
- `README.md` — user-facing docs (in Spanish)

## Game Architecture

**Board model** (`game.js`): 2D array `board[row][col]` where values are 0 (empty) or 1–7 (piece color index).

**Piece representation**: Objects with `{ type, shape, x, y }`. Shape is a 3×3 or 4×4 matrix. X/Y are position in board cells.

**Game loop** (`loop`): `requestAnimationFrame`-based. Accumulates elapsed time (`dropAccum`). When `dropAccum >= dropInterval`, piece falls one row. Piece locks when collision detected.

**Collision detection** (`collide`): Tests if piece shape at (x, y) overlaps board bounds or existing blocks. Used for movement, rotation, and line-clear logic.

**Rotation** (`tryRotate`): Rotates piece 90° CW via matrix transpose + row reversal. Tries wall-kick offsets `[0, ±1, ±2]` to fit rotated piece if initial rotation collides.

**Piece lock** (`lockPiece`): Merges current piece into board, clears complete lines, spawns next piece.

**Line clear** (`clearLines`): Scans board bottom-to-top. Each complete row is spliced out and empty row inserted at top. Updates score/level/dropInterval.

**Scoring**: Tetris (4 lines) = 800 × level. Hard drop +2 per cell. Soft drop +1 per row. Level increases every 10 lines; speed accelerates.

## Key Constants (in `game.js`)

| Const | Value | Purpose |
|-------|-------|---------|
| `COLS` | 10 | Board width in blocks |
| `ROWS` | 20 | Board height in blocks |
| `BLOCK` | 30 | Pixel size per block |
| `COLORS` | Array | RGB hex for pieces I–L (1–7); index 0 unused |
| `PIECES` | Array | 7 piece shapes as 3×3/4×4 matrices |
| `LINE_SCORES` | [0,100,300,500,800] | Points per 1–4 cleared lines |

Tweak `COLS`, `ROWS`, `BLOCK` to resize board, but update `<canvas width height>` in `index.html` accordingly. `BLOCK * COLS` = canvas width; `BLOCK * ROWS` = canvas height.

## Input Handling

Single `keydown` event listener covers movement, rotation, hard/soft drop, pause. Pause works during gameplay; restart only active on Game Over.

## Rendering Pipeline

1. `draw()` — clears canvas, draws grid, board blocks, ghost piece, current piece
2. `drawNext()` — renders next piece on separate canvas
3. `drawBlock()` — utility to paint a single block with highlight (white top stripe)
4. Grid lines via direct canvas stroking

Ghost piece calculated real-time in `ghostY()` during draw; does not cache.

## Common Edits

- **Add feature** (e.g. held piece, score multiplier): state variable in `init()`, logic in game loop or input handler
- **Adjust difficulty** (speed/scores): tweak `LINE_SCORES`, base `dropInterval`, or level-speed formula in `clearLines()`
- **Cosmetic** (colors, block style): edit `COLORS`, `drawBlock()` rendering
- **Board size**: change `COLS`/`ROWS`/`BLOCK`, sync canvas dimensions

All game state (`board`, `current`, `next`, `score`, `lines`, `level`, etc.) is global. No classes or modules; functional style.
