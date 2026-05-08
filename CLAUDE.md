# Checkers

A single-file browser-based 10×10 International Checkers game with AI opponent, local multiplayer, and online play.

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

1. **CSS** (`<style>`) — dark wood theme, settings modal, online overlay, chat area, landscape layout via media query
2. **HTML** (`<body>`) — settings overlay, online lobby overlay, sidebar (HUD/controls/chat), canvas board
3. **JavaScript** (`<script>`) — game logic, AI, rendering, Firebase online multiplayer

### Key JavaScript sections (in order)

| Section | What it does |
|---|---|
| `WORKER_SRC` string + `new Worker(blob)` | AI minimax runs in a Web Worker so the UI never freezes |
| Sound functions (`playTone`, `sndMove`, …) | Synthesized audio via Web Audio API — no audio files needed |
| Settings (`setMode`, `setLevel`, `setSound`) | Mutate `gameMode`, `aiDepth`, `soundEnabled` globals |
| Draw detection (`posHistory`, `noCaptureHalf`) | Threefold repetition + 50-move rule |
| King flash (`kingFlashes`, `flashTick`) | `requestAnimationFrame` overlay on promotion |
| Move logic (`getMoves`, `getCaptures`, `applyMove`) | Pure functions — take a board, return a new board |
| `handleBoardInput(x, y)` | Unified click + touch handler; flips coordinates for Black in online mode |
| `finishHumanTurn` → `finishAiTurn` / `setupFriendTurn` | Turn flow branches on `gameMode` |
| `draw()` + `drawPiece()` | Canvas renderer; applies `scale(-1,-1)` transform for Black's flipped view in online mode |
| Firebase / online section | Room creation/join, move sync, chat, rematch |

### Board representation

```js
board[row][col] = null | { color: 'w' | 'b', king: boolean }
```

Row 0 is the top (Black's back row). Row 9 is the bottom (White's back row).  
Only dark squares `(row + col) % 2 === 0` ever hold pieces.

### Game modes

- **`gameMode = 'ai'`** — White is human, Black is the AI Worker. W/D/L stats are tracked.
- **`gameMode = 'friend'`** — both colors are human (pass-and-play). No AI runs. Stats hidden.
- **`gameMode = 'online'`** — Firebase real-time multiplayer. Anonymous auth, 4-digit room code.

### AI

Minimax with alpha-beta pruning running inside a Web Worker (blob URL, no separate file).  
Depth is set by `aiDepth`: 1 (Easy), 3 (Medium), 7 (Strong).  
Evaluation weights: piece count, king value (9×), center control, advancement, back-row defense.

### History / undo

`boardHistory` is an array of `{ bd, t }` snapshots pushed before each human move.  
Undo in AI mode always resets `turn` to `'w'`; in friend mode it restores the saved turn.

### Online multiplayer (Firebase)

- Firebase Realtime Database + anonymous auth. No sign-up required.
- **Flow:** Player A creates game → gets 4-digit room code → Player B joins with code → game starts.
- **Firebase paths:**
  - `/rooms/{code}/moves/` — move objects pushed by each player; opponent listens via `child_added`
  - `/rooms/{code}/messages/` — chat messages (`{role, text, ts}`)
  - `/rooms/{code}/rematch/` — flags set when each player requests rematch
- **Board orientation:** Black player's canvas is flipped (`translate+scale(-1,-1)`) so their pieces always appear at the bottom. Click coordinates are mirrored accordingly.
- **Rematch:** roles (White/Black) alternate each rematch so the first-move advantage switches.
- **Chat:** message box appears in sidebar during online games; syncs in real time via Firebase.
- **Disconnect:** `onDisconnect().update({status:'abandoned'})` handles tab closes.

## Rules implemented

- 10×10 International Draughts board
- Mandatory captures (must-capture rule enforced)
- Multi-jump capture chains (human step-by-step; AI animated)
- Kings move and capture in all four diagonal directions, any distance (flying kings)
- Flying kings cannot reverse direction on the same diagonal within a capture chain
- King promotion with gold flash animation
- Draw: threefold repetition or 50 half-moves without a capture

## Settings (in-game modal)

| Setting | Values | Notes |
|---|---|---|
| Difficulty | Easy / Medium / Strong | AI mode only; takes effect next New Game |
| Sound Effects | On / Off | Immediate |
| Game Mode | vs Computer / vs Friend / Online | Takes effect next New Game |

## Roadmap

### Next: Native app (Capacitor)

- Install Node.js + Capacitor CLI
- `npx cap init` → `npx cap add ios` + `npx cap add android`
- Copy `index.html` into the Capacitor `www/` folder
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
