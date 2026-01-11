# ToeTactic

ToeTactic is a retro-styled misère Tic-Tac-Toe variant where _making a line means you lose_. The single-page app lives entirely in `index.html` and ships with:

- Local multiplayer for 2–4 players with custom emoji pieces.
- Bot battles with four AI difficulties, including an “impossible” minimax solver.
- Theme switcher (Vista, Windows 98, Mac OS Aqua) plus dark-mode support.
- Full-screen effects (dynamic shadows, particles, confetti, ripple cursor) and adaptive music.

## Running Locally

1. Serve the repo root with any static HTTP server (`python3 -m http.server` works).
2. Open `http://localhost:8000/index.html` in a modern browser (desktop recommended).
3. Allow audio playback in the browser if prompted (autoplay is needed for menu music).

_No build tooling is required; the experience is 100% client-side._

## UI, Components, and Game Flow

- **Main Window (`.window`)** – Vista/Win98/Aqua-themed frame that hosts every view.
- **Menu (`#menu`)** – Entry point with `Play With Bot`, `Play Local`, `How To Play`, and `Settings`.
- **Settings (`#settingsMenu`)** – Controls grid size (3–10), AI difficulty, per-player emojis, player count, theme, and audio sliders.
- **Board Container (`#boardContainer`)** – Visible during a match; includes the turn indicator, player queue, board grid, calculation terminal, and minimal game menu.
- **Instructions Panel (`#instructions`)** – Animated walkthrough shown from the menu.
- **Modal (`#gameModal`)** – Shared overlay used for win/lose/draw states with “Play Again” and “Main Menu” actions.
- **Audio Elements** – Injected dynamically (`titleMusic`, `botMusic`, `localMusic`) and controlled through the master/music volume sliders and toolbar toggle.

Flow overview:

1. **Menu → Settings** (optional) – configure players, theme, and audio.
2. **Menu → Play With Bot / Play Local** – `startGame()` builds the board, player queue, and music context.
3. **Gameplay** – `makeMove()` enforces turns, checks for losing lines via `checkLose()`, and drives the bot via `botMove()`.
4. **Game End** – `showGameModal()` announces the loser/winner/draw and allows replay or returning to the menu.

## Public API & Function Reference

Every function below is declared in the `<script>` block of `index.html` and can be reused or extended from the browser console or additional scripts.

### Initialization & Configuration

- **`initializeAudioSettings()`** – Syncs sliders/toggles with `localStorage`, wires listeners, and updates volume envelopes.
- **`addAudioElements()`** – Injects hidden `<audio>` tags for `game title.mp3`, `game bot.mp3`, and `game local.mp3`, priming title music.
- **`initializeThemeSelector()`** – Restores the persisted theme (`default`, `win98`, `aqua`), listens for dropdown changes, and animates transitions.
- **`initializeDynamicShadows()`** – Adds a document-level mousemove listener to tilt/illuminate each `.cell` based on the cursor position.
- **`createParticles()` / `createParticle(container)`** – Spawns and maintains animated background particles behind the glass window.
- **`addNotificationDot()`** – Adds a pulsing indicator to the “How To Play” button until the user opens the guide.
- **`createWin98Icons()`** – Rebuilds title bar buttons with Win98 chrome whenever `data-theme="win98"` is active.

### Menu & View Management

- **`openMenu(element)`** – Hides all views, reveals the supplied panel (`menu`, `settingsMenu`, or `instructions`), updates the window title, and switches back to title music.
- **`closeAllMenus()`** – Hides menu, settings, board, instructions, and the in-game button bar.
- **`updateGameTitle()`** – Writes “ToeTacTic – vs Bot (Difficulty)” or `ToeTacTic – N Players` into the title bar according to mode.
- **`updateTheme(mode)`** – Applies/removes the `dark-mode` class based on saved preference or OS theme listener.
- **`addMusicToggle()`** – Rebuilds toolbar controls for the current theme; toggles `audioEnabled` and persists it when clicked.

### Game Setup & State

- **`updateEmojiInputs()`** – Regenerates emoji fields to match the selected player count, seeding defaults for players 1–4.
- **`startGame()`** – Central entry point: closes menus, configures board dimensions, seeds players (human or bot), randomizes the opener, builds grid cells, restarts music, resets the terminal, and kicks off shadows/animations.
- **`createCell(row, col)`** – Builds a clickable `.cell` wrapper with a `.cell-content` child, storing row/column metadata and entrance animation delay.
- **`handleClick(event)`** – Guards against clicks on filled cells or during the bot’s turn, then calls `makeMove()` with the clicked cell.
- **`makeMove(cell)`** – Places the current player’s emoji, marks the tile as placed, evaluates lose/draw states, advances turns, updates the UI, and schedules bot responses when needed.
- **`updateTurnIndicator()`** – Animates the `#turnIndicator` with the latest player emoji using slide-in/out transitions.
- **`updatePlayerQueue()`** – Rebuilds the player-card grid, highlighting the active player and preserving animation offsets.
- **`resetBoard()`** – Clears every cell, removes win highlights, resets the player index, and refreshes UI indicators.
- **`updateEmojiInputs()`** – (duplicate mention) ensures the DOM inputs mirror the configured player count; call after changing `#playerCount`.

### Win / Lose Evaluation

- **`isBoardFull()`** – Returns `true` when every `.cell-content` is non-empty, signaling a draw if no lose condition triggered.
- **`checkLose(row, col, visualUpdate = true)`** – Evaluates the four line directions around the last move, optionally adding highlight classes, and returns `true` if the active player just formed three in a row (thus losing).
- **`checkDirection(row, col, rowDir, colDir, visualUpdate = true)`** – Shared helper that traverses both directions to count consecutive symbols and flag win cells.
- **`showLosingStreak(row, col)`** – Clears previous highlights and re-runs `checkDirection` for every axis to accentuate the doomed line.
- **`checkGameState()`** – Lightweight status probe used by the bot to detect loses/draws without changing the UI.

### Bot & AI

- **`botMove()`** – Handles animated “thinking” state, enforces a 20s timeout (bot forfeit), and dispatches the correct move finder per difficulty.
- **`botMoveImpossible()`** – Convenience wrapper that executes the perfect-play variant when the “Impossible” option is selected.
- **`findBestMove()`** – Depth-limited (3-ply) minimax with alpha-beta pruning to choose strong moves for Easy/Medium/Hard bots.
- **`findBestMoveImpossible()`** – Extended (6-ply) minimax pass that streams analysis lines to the calculation terminal for transparency.
- **`makeRandomMove()`** – Picks a random empty cell, used to add human-like mistakes on easier difficulties.
- **`minimax(board, depth, isMaximizing, alpha, beta, maxDepth)`** – Core solver that simulates future moves, returns positive scores when the human is closer to forcing the bot to lose, and supports pruning.
- **`evaluateBoard(board)`** & **`evaluateLine(cells)`** – Heuristics for partial lines, rewarding setups that push the opponent toward completing three.
- **`getRow(row)`**, **`getColumn(col)`**, **`getDiagonal()`**, **`getAntiDiagonal()`** – Extractors that feed `evaluateLine`.
- **`addTerminalLine(text)`** – Streams timestamped analysis snippets to the faux terminal, keeping only the latest 50 entries.

### Audio & Effects

- **`playMusic(id)`** – Crossfades between the three audio tracks (`titleMusic`, `botMusic`, `localMusic`) and resets others.
- **`fadeAudio(audio, targetVolume, duration = 1000)`** – Smoothly animates volume changes using an ease-in-out curve.
- **`updateAudioVolumes()`** – Applies the current `masterVolume` × `musicVolume` mix to every track and keeps the toolbar button in sync.
- **`startContinuousConfetti()`**, **`stopConfetti()`**, **`createConfettiWave()`** – Drive the celebratory confetti bursts after a player loses.
- **`startContinuousConfetti()`** also clears previous intervals to avoid duplicate particle storms.
- **`createParticles()` / `createParticle()`** – (See Initialization section) maintain the animated ambient background.
- **`addAudioElements()`** – (See Initialization section) injects the `<audio>` tags once on load.

### Modal & Overlay Management

- **`showGameModal(title, content)`** – Configures the result modal, wires “Play Again”/“Main Menu” callbacks, pauses confetti resets, and animates the overlay in.
- **`hideModal()`** – Reverses the modal animation and hides the overlay after the transition completes.

## Usage Examples

### 1. Starting a Custom Four-Player Match

```js
// From the devtools console:
playerCountInput.value = 4;
updateEmojiInputs();
document.querySelectorAll('.emoji-input').forEach((input, i) => {
  input.value = ['❌','⭕','⚠️','💣'][i];
});
gridSizeInput.value = 6; // required for 4 players
startGame();
```

### 2. Forcing the Bot to Reveal Its Next Move

```js
// Only available after `startGame()` with Play With Bot
const bestCell = findBestMoveImpossible();
bestCell && bestCell.classList.add('hint');
```

### 3. Applying a Theme and Muting Audio Programmatically

```js
document.documentElement.setAttribute('data-theme', 'aqua');
initializeThemeSelector();         // rebuild controls for the new theme
audioEnabled = false;
updateAudioVolumes();              // immediately fade out every track
addMusicToggle();                  // refresh toolbar icons
```

## Extending ToeTactic

- **New audio tracks** – Add `<source>` elements to `addAudioElements()` or swap the MP3 file names; reuse `playMusic()` for consistent crossfades.
- **Custom themes** – Extend the CSS `:root[data-theme="..."]` rules and update the dropdown options in `#themeSelector`.
- **Alternative victory conditions** – Modify `checkLose()` thresholds (currently hardcoded for “three in a row”) or let `gridSize` dictate the streak length.
- **Remote multiplayer prototype** – Hook into `makeMove()` to broadcast local moves and feed remote updates back through the same function for perfect parity.

Because the app is framework-free, every function, DOM element, and style token is open for extension with zero build steps—just edit `index.html`, refresh the browser, and iterate.
