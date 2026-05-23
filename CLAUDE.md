# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-file static web app: a Snake game with an NYC Subway theme, built as a promotional landing page for the music release *Stroopwafel (KITA Remix)* by Desert Vision. It is hosted on GitHub Pages at `stroopwafel.desertvision.it` (configured via `CNAME`).

## Repository Structure

There is **no build step, no package manager, and no framework**. The entire application lives in `index.html`:

- `index.html` — All HTML, CSS (in `<style>`), and JavaScript (in `<script>`) in one file
- `stroopwafel_kita.mp3` — Audio track; short snippets play when the snake eats a waffle
- `og-image.png` — Open Graph / Twitter card image for social sharing
- `CNAME` — GitHub Pages custom domain (`stroopwafel.desertvision.it`)

## Development & Deployment

Since there is no build toolchain, development is direct file editing:

```bash
# Serve locally (any static file server works)
python3 -m http.server 8080
# then open http://localhost:8080
```

Deployment is automatic: push to the default branch and GitHub Pages serves the result. No CI pipeline exists.

## Architecture Inside `index.html`

### Rendering
- Uses the HTML5 Canvas API (`<canvas id="gc">`).
- `resize()` computes grid dimensions (`COLS`, `ROWS`) from the canvas container width. `CELL = 20px`.
- `drawBg()` renders stylized subway route lines using `LINES` and `getBgLines()` — the background is purely decorative.
- `drawWaffle()`, `drawSnake()`, `drawEnemySnake()` handle all in-game sprite rendering.

### Game Loop
- Driven by `requestAnimationFrame` inside `loop(ts)`.
- Speed is controlled by `getInterval()`, which returns a millisecond delay from `SPEED_TABLE` based on the current `speed` level (1–10). Speed increases every 80 points.
- `update()` advances snake position, checks collisions, handles food collection, and triggers enemy movement every `ENEMY_INTERVAL` (4) ticks.

### State
All game state is module-level variables:
- `snake` — array of `{x, y}` segments; head is `snake[0]`
- `dir` / `nextDir` — current/queued direction; buffered to prevent 180° reversal
- `enemies` — array of enemy snake objects `{body, dir, color, turnCountdown}`
- `waffleMarkers` — tracks segment indices that display a waffle icon on the snake body
- `food` — `{x, y}` position of the current collectible waffle
- `score`, `speed`, `highScore`, `running`, `gameOver`

### Enemy Snakes
- Spawn at score thresholds defined in `ENEMY_SPAWN_SCORES` (`[50, 100, 150]`).
- Move via `moveEnemies()`: random turns when hitting a wall or when `turnCountdown` reaches zero.
- Collision with any enemy segment ends the game.

### Audio
- `<audio id="track">` loads `stroopwafel_kita.mp3`.
- `playSnippet()` picks a random offset in the track and plays ~4.5 seconds; called on each waffle eaten.

### Leaderboard (Supabase)
- Backed by a Supabase project at `SB_URL`. The `SB_KEY` in the source is an **anon/public key** — this is intentional for a client-side-only app.
- `submitScore(name, pts)` — POSTs `{name, score}` to the `scores` table.
- `fetchLeaderboardOverlay()` — GETs top 5 scores; displayed on the game-over screen.
- The `scores` table schema expected: columns `name` (text) and `score` (integer).

### Controls
- Keyboard: Arrow keys via `keydown` listener.
- Mobile: Fixed D-pad at the bottom of the viewport (CSS `position: fixed`), handled via `touchstart` and `mousedown` listeners on `.dpad-btn` elements.

## Key Constants to Know

| Constant | Value | Purpose |
|---|---|---|
| `CELL` | `20` | Grid cell size in pixels |
| `ENEMY_INTERVAL` | `4` | Enemy moves every N game ticks |
| `ENEMY_SPAWN_SCORES` | `[50, 100, 150]` | Score thresholds for enemy spawns |
| `SPEED_TABLE` | `[220…92]` ms | Tick interval per speed level |
| `PRESAVE_URL` | hypeddit link | Pre-save CTA shown on game-over screen |
