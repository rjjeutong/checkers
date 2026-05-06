# Checkers

A single-file browser-based 10×10 International Checkers game with AI opponent and local multiplayer.

## Project structure

```
index.html   — the entire app: HTML, CSS, and JavaScript in one file
```

No build step, no dependencies, no package manager. Open `index.html` directly in a browser.

## Architecture

Everything lives inside `index.html` in three parts:

1. **CSS** (`<style>`) — dark wood theme, settings modal, landscape layout via media query
2. **HTML** (`<body>`) — settings overlay, sidebar (HUD/controls), canvas board
3. **JavaScript** (`<script>`) — game logic, AI, rendering

### Key JavaScript sections (in order)

| Section | What it does |
|---|---|
| `WORKER_SRC` string + `new Worker(blob)` | AI minimax runs in a Web Worker so the UI never freezes |
| Sound functions (`playTone`, `sndMove`, …) | Synthesized audio via Web Audio API — no audio files needed |
| Settings (`setMode`, `setLevel`, `setSound`) | Mutate `gameMode`, `aiDepth`, `soundEnabled` globals |
| Draw detection (`posHistory`, `noCaptureHalf`) | Threefold repetition + 50-move rule |
| King flash (`kingFlashes`, `flashTick`) | `requestAnimationFrame` overlay on promotion |
| Move logic (`getMoves`, `getCaptures`, `applyMove`) | Pure functions — take a board, return a new board |
| `handleBoardInput(x, y)` | Unified click + touch handler for both game modes |
| `finishHumanTurn` → `finishAiTurn` / `setupFriendTurn` | Turn flow branches on `gameMode` |
| `draw()` + `drawPiece()` | Canvas renderer, called after every state change |

### Board representation

```js
board[row][col] = null | { color: 'w' | 'b', king: boolean }
```

Row 0 is the top (Black's back row). Row 9 is the bottom (White's back row).  
Only dark squares `(row + col) % 2 === 0` ever hold pieces.

### Game modes

- **`gameMode = 'ai'`** — White is human, Black is the AI Worker. W/D/L stats are tracked.
- **`gameMode = 'friend'`** — both colors are human (pass-and-play). No AI runs. Stats hidden.

### AI

Minimax with alpha-beta pruning running inside a Web Worker (blob URL, no separate file).  
Depth is set by `aiDepth`: 1 (Easy), 3 (Medium), 7 (Strong).  
Evaluation weights: piece count, king value (9×), center control, advancement, back-row defense.

### History / undo

`boardHistory` is an array of `{ bd, t }` snapshots pushed before each human move.  
Undo in AI mode always resets `turn` to `'w'`; in friend mode it restores the saved turn.

## Rules implemented

- 10×10 International Draughts board
- Mandatory captures (must-capture rule enforced)
- Multi-jump capture chains (human step-by-step; AI animated)
- Kings move and capture in all four diagonal directions, any distance (flying kings)
- King promotion with gold flash animation
- Draw: threefold repetition or 50 half-moves without a capture

## Settings (in-game modal)

| Setting | Values | Notes |
|---|---|---|
| Difficulty | Easy / Medium / Strong | AI mode only; takes effect next New Game |
| Sound Effects | On / Off | Immediate |
| Game Mode | vs Computer / vs Friend / Online | Online is a placeholder; takes effect next New Game |

## Planned / future work

- Native iOS & Android packaging (Capacitor)
- App Store / Google Play icons and splash screens
- Online multiplayer (requires a backend or a real-time service like Firebase or Supabase)
- Move log / game notation
- Per-turn timer

## Development

No tooling required — just edit `index.html` and refresh the browser.  
For mobile testing, serve locally with any static file server, e.g.:

```bash
npx serve .
# or
python -m http.server 8080
```

Then open the local URL on your phone.
