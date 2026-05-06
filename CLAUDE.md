# Checkers

A single-file browser-based 10×10 International Checkers game with AI opponent and local multiplayer.

## Live deployment

- **GitHub repo:** https://github.com/rjjeutong/checkers
- **GitHub Pages (live):** https://rjjeutong.github.io/checkers/
- Branch: `master` · root folder · deployed via GitHub Pages (repo must stay public)

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
- **`gameMode = 'online'`** — placeholder in Settings UI; not yet implemented (see roadmap).

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

## Roadmap

### Next: Online multiplayer (Firebase)

Agreed approach — Firebase Realtime Database, anonymous auth, no sign-up required.

**Player flow:**
1. Player A taps "Create game" → receives a 4-digit room code
2. Player B taps "Join game" → enters the code
3. Moves sync in real time via Firebase
4. No accounts needed — anonymous sessions only

**Why Firebase:**
- Free tier covers casual game traffic easily
- Works inside a Capacitor WebView without native plugins
- Realtime Database is simple to wire to the existing board state

**New file needed:** `firebase.js` (or inline in `index.html`) — initialise app, write moves to `/rooms/{code}/moves`, listen for opponent moves.

**App store impact:** None blocking. Adds one requirement: a Privacy Policy page (needed anyway for store submission). Use a free generator (e.g. privacypolicygenerator.info).

### After that: Native app (Capacitor)

- Install Node.js + Capacitor CLI
- `npx cap init` → `npx cap add ios` + `npx cap add android`
- Copy `index.html` (and `firebase.js`) into the Capacitor `www/` folder
- Generate icons and splash screens with `@capacitor/assets`
- Build iOS in Xcode (requires a Mac) · Build Android in Android Studio
- Submit to App Store and Google Play

### Remaining nice-to-haves

- Move log / game notation
- Per-turn timer
- Privacy Policy page (required before store submission)

## Development

No tooling required — just edit `index.html` and refresh the browser.  
For mobile testing, serve locally with any static file server, e.g.:

```bash
npx serve .
# or
python -m http.server 8080
```

Then open the local URL on your phone.
