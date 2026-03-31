# AUREUM Snake Game Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a fully playable, premium mobile-first snake game with Art Deco aesthetic in a single self-contained HTML file.

**Architecture:** Single HTML file with inline CSS and JS. No framework, no build step. Game rendered with absolutely positioned divs (not canvas). Game loop via `requestAnimationFrame` with tick accumulator. Audio via Web Audio API synthesis.

**Tech Stack:** HTML/CSS/JS, Google Fonts (Playfair Display + Outfit), Web Audio API, localStorage

**Spec:** `docs/superpowers/specs/2026-03-30-aureum-snake-game-design.md`

---

## File Structure

This is a single-file project:

- **Create:** `aureum-snake.html` — the complete game

---

### Task 1: HTML Shell + CSS Foundation

Build the static HTML structure and all CSS. No JS yet — just a pixel-perfect Art Deco layout with placeholder content showing the start screen state.

**Files:**
- Create: `aureum-snake.html`

- [ ] **Step 1: Create the HTML file with head, fonts, CSS variables, and reset**

Create `aureum-snake.html` with:
- DOCTYPE, meta charset, viewport meta (with `user-scalable=no`)
- Google Fonts `<link>` for Playfair Display (400, 700, 900, italic 400) and Outfit (200, 300, 400, 500)
- CSS reset (`*, *::before, *::after { margin:0; padding:0; box-sizing:border-box; }`)
- `:root` with full color palette from spec Section 1.1
- `--cell: 18px` CSS variable
- Base `html, body` styles: height 100%, overflow hidden, obsidian background, text-light color, Outfit font, weight 300, tap-highlight transparent, user-select none

- [ ] **Step 2: Add background decorative elements CSS and HTML**

Add CSS and HTML for:
- `.bg-deco`: fixed, radial gold gradients at top (6% opacity) and bottom (4% opacity)
- `.deco-line-top` / `.deco-line-bottom`: fixed, centered, 1px wide, 60px tall, gold linear-gradient to transparent, 20% opacity
- `.side-deco.left` / `.side-deco.right`: fixed at viewport midpoint, 1px wide, 200px tall, gold fade, 12% opacity

- [ ] **Step 3: Add the app container with two-screen structure and header**

The HTML uses two screen wrappers from the start — `#menuScreen` and `#gameScreen` — inside the `.app` container. Only one is visible at a time. The game board, shimmer frame, and overlay live inside `#gameScreen`. The menu screen has its own copy of the board area (empty, decorative only) or shares the board via DOM manipulation in JS.

**Approach:** Both screens live inside `.app`. `#menuScreen` contains: header (title/ornament), empty decorative board, record display, play button. `#gameScreen` contains: score bar, game board with overlay, rank bar, controls, bottom bar. `#gameScreen` starts with `display: none`.

Add:
- `.app` container: relative, z-index 10, max-width 420px, centered, full height, flex column, padding 16px 20px with `env(safe-area-inset-bottom)`
- `#menuScreen` wrapper div (flex column, flex 1)
- `#gameScreen` wrapper div (flex column, flex 1, `display: none`)
- Inside `#menuScreen` — `.header` with centered layout:
  - `.deco-ornament`: flex row with two `.ornament-line` (40px, gold gradient, 40% opacity) flanking `.ornament-diamond` (6px, gold, rotated 45deg, 50% opacity)
  - `.title`: Playfair Display 900, 26px, letter-spacing 10px, gold color with text-shadow glow — text content: "AUREUM"
  - `.subtitle`: 9px, letter-spacing 4px, uppercase, text-dim color — text content: "SERPENTIS LUDUS"
- Speaker icon button (`id="soundBtn"`, top-right absolute): SVG speaker icon, 36x36px, charcoal bg, gold 10% border

- [ ] **Step 3b: Add menu screen record display and play button**

Below the decorative board in `#menuScreen`, add:
- `.menu-record` (`id="menuRecord"`): Playfair Display italic, 13px, gold, 60% opacity, centered, padding 16px 0 — text: "Record: 0"
- `.menu-play-btn` (`id="menuPlayBtn"`): `.deco-btn.gold` style, centered, Outfit 11px, letter-spacing 4px, uppercase — text: "PLAY"

- [ ] **Step 4: Add score display bar CSS and HTML (inside `#gameScreen`)**

Add `.score-display` inside `#gameScreen` with:
- Flex row, gold 10% border, relative positioning
- `.score-corner-tl` / `.score-corner-br`: 8px absolute corner accents in gold, 30% opacity
- Three `.score-section` children (flex: 1, padding 14px 16px, centered text)
- First section gets `.featured` class (gold 4% background)
- `.score-divider` on 2nd and 3rd sections (absolute, left 0, top/bottom 20%, 1px, gold 10%)
- `.score-label`: 8px, letter-spacing 3px, uppercase, text-dim
- `.score-value`: Playfair Display 700, 26px, text-light (featured gets gold color + text-shadow)
- Add `id` attributes: `id="scoreValue"` on first score value, `id="recordValue"` on second, `id="chainValue"` on third
- Set static content: Score 0, Record 0, Chain 3

- [ ] **Step 5: Add game board with shimmer border**

Add:
- `.game-container`: flex 1, centered, min-height 0
- `.game-frame`: 3px padding, gold shimmer gradient background (`linear-gradient(135deg, gold, gold-dark, gold, gold-dark)`), `background-size: 200% 200%`, `@keyframes goldShimmer` animating background-position 0%→100%→0% over 6s
- `.game-frame-inner`: 8px padding, obsidian background
- `.game-board` (`id="gameBoard"`): relative, `width: calc(18 * var(--cell))`, `height: calc(18 * var(--cell))`, surface background, overflow hidden
- `.deco-grid`: absolute inset, background-image with gold 3% grid lines at cell intervals
- Four `.board-corner-deco` elements: 20px, gold 2px borders on two sides, 8% opacity

- [ ] **Step 6: Add rank bar, D-pad controls, and bottom bar**

Add:
- `.rank-bar`: flex row, gap 12px, margin 14px 0 10px
  - `.rank-label`: Playfair Display italic, 11px, gold, 60% opacity
  - `.rank-track`: flex 1, 2px height, gold 8% background
  - `.rank-fill` (`id="rankFill"`): height 100%, width 0%, gold gradient, gold glow shadow
  - `.rank-title` (`id="rankTitle"`): Playfair Display 700, 11px, gold, letter-spacing 2px — content "I"
- `.controls`: flex column centered, gap 4px, padding 12px 0 6px
  - Two `.controls-row` (flex, gap 4px) with `.ctrl-btn` buttons (62x62px, charcoal bg, gold 10% border)
  - Row 1: spacer, up arrow (`data-dir="up"`), spacer
  - Row 2: left (`data-dir="left"`), down (`data-dir="down"`), right (`data-dir="right"`)
  - Each button has `.ctrl-inner-border` (absolute, inset 3px, gold 4% border) and SVG arrow icon (18px, stroke-width 1.5)
  - Active state CSS: gold 8% bg, gold 25% border, scale 0.93, gold icon color
- `.bottom-bar`: flex, space-between, padding 8px 0 4px
  - Pause button (`id="pauseBtn"`, `.deco-btn.ghost`): Outfit, 11px, letter-spacing 4px, uppercase, ghost style
  - Play button (`id="playBtn"`, `.deco-btn.gold`): gold gradient background, obsidian text, box-shadow

- [ ] **Step 7: Add entry animations and responsive media queries**

Add:
- `@keyframes decoReveal`: from `opacity:0; translateY(8px)` to `opacity:1; translateY(0)`
- Apply with staggered delays: header 0.2s, score 0.4s, board 0.6s, rank 0.75s, controls 0.85s, bottom 1.0s
- All animated elements start with `opacity: 0`
- `@media (max-height: 700px)`: reduced padding, smaller stat values (20px), smaller buttons (54px), compressed margins
- `@media (min-width: 421px)`: `--cell: 22px`

- [ ] **Step 8: Add overlay CSS for pause and game over states**

Add:
- `.overlay` (`id="overlay"`): absolute inset on game board, obsidian 85% opacity, `backdrop-filter: blur(8px)`, flex column centered, gap 20px, border-radius matching board, z-index 20, `display: none`
- `.overlay.visible`: `display: flex`
- `.overlay-title` (`id="overlayTitle"`): Playfair Display 900, 24px, gold, letter-spacing 8px
- `.overlay-subtitle` (`id="overlaySubtitle"`): Outfit, 11px, text-dim, letter-spacing 3px
- `.overlay-score` (`id="overlayScore"`): Playfair Display 700, 36px, gold (for game over)
- `.overlay-record` (`id="overlayRecord"`): gold pulse keyframe animation for new record label
- Primary button (`id="overlayBtnPrimary"`): `.deco-btn.gold` style
- Secondary button (`id="overlayBtnSecondary"`): `.deco-btn.ghost` style

- [ ] **Step 9: Verify the static layout renders correctly**

Open `aureum-snake.html` in browser. Verify:
- Art Deco styling matches spec — gold on obsidian, shimmer border animating, ornaments visible
- Start screen shows title, empty board, play button
- Score bar shows 0 / 0 / 3
- D-pad arrows render with correct layout
- Entry animations play on load
- Responsive: resize to <700px height (compressed), >420px width (larger cells)

- [ ] **Step 10: Commit**

```bash
git add aureum-snake.html
git commit -m "feat: AUREUM snake game — static HTML/CSS layout with Art Deco styling"
```

---

### Task 2: Game Engine — Core Snake Logic

Add the JavaScript game engine: snake state, movement, food spawning, collision detection, scoring, levels. No rendering yet — just the pure game state logic wired to a game loop.

**Files:**
- Modify: `aureum-snake.html` (add `<script>` before `</body>`)

- [ ] **Step 1: Add game state variables and constants**

Add a `<script>` tag before `</body>` with:
```javascript
// Constants
const GRID = 18;
const INITIAL_LENGTH = 3;
const FOOD_PER_LEVEL = 5;
const MAX_LEVEL = 10;
const BASE_TICK = 150;
const TICK_DECREASE = 8;
const MIN_TICK = 78;

// Game state
let snake = [];
let direction = { x: 1, y: 0 };
let directionQueue = [];
let food = { x: 0, y: 0 };
let score = 0;
let highScore = parseInt(localStorage.getItem('aureum-snake-highscore')) || 0;
let level = 1;
let foodEaten = 0;
let gameState = 'menu'; // 'menu' | 'playing' | 'paused' | 'gameover'
let lastTick = 0;
let tickAccumulator = 0;
```

- [ ] **Step 2: Add game initialization and food spawn functions**

```javascript
function initGame() {
  snake = [];
  for (let i = 0; i < INITIAL_LENGTH; i++) {
    snake.push({ x: 9 - i, y: 9 });
  }
  direction = { x: 1, y: 0 };
  directionQueue = [];
  score = 0;
  level = 1;
  foodEaten = 0;
  tickAccumulator = 0;
  spawnFood();
}

function spawnFood() {
  const occupied = new Set(snake.map(s => s.x + ',' + s.y));
  let pos;
  do {
    pos = {
      x: Math.floor(Math.random() * GRID),
      y: Math.floor(Math.random() * GRID)
    };
  } while (occupied.has(pos.x + ',' + pos.y));
  food = pos;
}
```

- [ ] **Step 3: Add tick/update function with movement, collision, eating**

```javascript
function getTickInterval() {
  return Math.max(MIN_TICK, BASE_TICK - (level - 1) * TICK_DECREASE);
}

function update() {
  if (directionQueue.length > 0) {
    direction = directionQueue.shift();
  }
  const head = { x: snake[0].x + direction.x, y: snake[0].y + direction.y };

  // Wall collision
  if (head.x < 0 || head.x >= GRID || head.y < 0 || head.y >= GRID) {
    gameOver();
    return;
  }

  // Self collision
  if (snake.some(s => s.x === head.x && s.y === head.y)) {
    gameOver();
    return;
  }

  snake.unshift(head);

  // Eat food
  if (head.x === food.x && head.y === food.y) {
    score += 10 * level;
    foodEaten++;
    if (foodEaten % FOOD_PER_LEVEL === 0 && level < MAX_LEVEL) {
      level++;
      onLevelUp();
    }
    spawnFood();
    onEatFood();
  } else {
    snake.pop();
  }
}
```

- [ ] **Step 4: Add direction change with 180-degree prevention**

```javascript
function setDirection(x, y) {
  const lastDir = directionQueue.length > 0 ? directionQueue[directionQueue.length - 1] : direction;
  // Prevent 180-degree reversal
  if (lastDir.x + x === 0 && lastDir.y + y === 0) return;
  // Max 2 queued inputs to prevent spamming
  if (directionQueue.length < 2) {
    directionQueue.push({ x, y });
  }
}
```

- [ ] **Step 5: Add game state transitions**

```javascript
function startGame() {
  initGame();
  gameState = 'playing';
  lastTick = performance.now();
  renderAll();
  updateUI();
}

function pauseGame() {
  if (gameState === 'playing') {
    gameState = 'paused';
    showOverlay('pause');
  }
}

function resumeGame() {
  if (gameState === 'paused') {
    gameState = 'playing';
    hideOverlay();
    lastTick = performance.now();
    tickAccumulator = 0;
  }
}

function gameOver() {
  gameState = 'gameover';
  const isNewRecord = score > highScore;
  if (isNewRecord) {
    highScore = score;
    localStorage.setItem('aureum-snake-highscore', String(highScore));
  }
  showOverlay('gameover', isNewRecord);
}

function returnToMenu() {
  gameState = 'menu';
  showMenu();
}

// Placeholder callbacks — will be implemented in rendering task
function onEatFood() {}
function onLevelUp() {}
function renderAll() {}
function updateUI() {}
function showOverlay(type, isNewRecord) {}
function hideOverlay() {}
function showMenu() {}
```

- [ ] **Step 6: Add the main game loop**

```javascript
function gameLoop(timestamp) {
  requestAnimationFrame(gameLoop);
  if (gameState !== 'playing') return;

  if (!lastTick) lastTick = timestamp;
  tickAccumulator += timestamp - lastTick;
  lastTick = timestamp;

  const interval = getTickInterval();
  while (tickAccumulator >= interval) {
    tickAccumulator -= interval;
    update();
    if (gameState !== 'playing') return; // game over happened
    renderAll();
    updateUI();
  }
}

requestAnimationFrame(gameLoop);
```

- [ ] **Step 7: Add Roman numeral helper**

```javascript
function toRoman(num) {
  const numerals = ['', 'I', 'II', 'III', 'IV', 'V', 'VI', 'VII', 'VIII', 'IX', 'X'];
  return numerals[num] || String(num);
}
```

- [ ] **Step 8: Commit**

```bash
git add aureum-snake.html
git commit -m "feat: add core snake game engine — state, movement, collision, scoring, levels"
```

---

### Task 3: DOM Rendering — Connect Game State to UI

Wire the game engine to the DOM: render snake segments and food as positioned divs, update score/rank/chain displays, manage screen transitions and overlays.

**Files:**
- Modify: `aureum-snake.html`

- [ ] **Step 1: Cache DOM references**

At the top of the script (after constants/state), add:
```javascript
const board = document.getElementById('gameBoard');
const scoreEl = document.getElementById('scoreValue');
const recordEl = document.getElementById('recordValue');
const chainEl = document.getElementById('chainValue');
const rankFill = document.getElementById('rankFill');
const rankTitle = document.getElementById('rankTitle');
const overlay = document.getElementById('overlay');
const overlayTitle = document.getElementById('overlayTitle');
const overlaySubtitle = document.getElementById('overlaySubtitle');
const overlayScore = document.getElementById('overlayScore');
const overlayRecord = document.getElementById('overlayRecord');
const overlayBtnPrimary = document.getElementById('overlayBtnPrimary');
const overlayBtnSecondary = document.getElementById('overlayBtnSecondary');
const menuScreen = document.getElementById('menuScreen');
const gameScreen = document.getElementById('gameScreen');
const menuRecord = document.getElementById('menuRecord');
const soundBtn = document.getElementById('soundBtn');
```

All `id` attributes were already added in Task 1.

- [ ] **Step 2: Implement renderAll() — snake and food rendering**

```javascript
function renderAll() {
  // Clear old snake/food elements
  board.querySelectorAll('.snake-segment, .food').forEach(el => el.remove());

  const cell = parseInt(getComputedStyle(document.documentElement).getPropertyValue('--cell'));

  // Render snake
  snake.forEach((seg, i) => {
    const el = document.createElement('div');
    const type = i === 0 ? 'head' : i === snake.length - 1 ? 'tail' : 'body';
    el.className = 'snake-segment ' + type;
    el.style.left = seg.x * cell + 'px';
    el.style.top = seg.y * cell + 'px';

    const inner = document.createElement('div');
    inner.className = 'gem';
    if (type === 'body') {
      inner.style.opacity = String(1 - (i / snake.length) * 0.5);
    }
    el.appendChild(inner);
    board.appendChild(el);
  });

  // Render food
  const foodEl = document.createElement('div');
  foodEl.className = 'food';
  foodEl.style.left = food.x * cell + 'px';
  foodEl.style.top = food.y * cell + 'px';
  const ruby = document.createElement('div');
  ruby.className = 'ruby';
  foodEl.appendChild(ruby);
  board.appendChild(foodEl);
}
```

- [ ] **Step 3: Implement updateUI() — scores, rank bar, chain**

```javascript
function updateUI() {
  scoreEl.textContent = score.toLocaleString('en-US');
  recordEl.textContent = highScore.toLocaleString('en-US');
  chainEl.textContent = snake.length.toLocaleString('en-US');

  // Rank bar
  const progress = (foodEaten % FOOD_PER_LEVEL) / FOOD_PER_LEVEL * 100;
  rankFill.style.width = progress + '%';
  rankTitle.textContent = toRoman(level);
}
```

- [ ] **Step 4: Implement score pop animation on change**

```javascript
function animateScorePop() {
  scoreEl.style.transform = 'scale(1.15)';
  setTimeout(() => { scoreEl.style.transform = 'scale(1)'; }, 150);
}

// Update onEatFood:
function onEatFood() {
  animateScorePop();
  playSound('eat');
  vibrate(30);
}
```

- [ ] **Step 5: Implement screen transitions — menu ↔ gameplay**

The `#menuScreen` and `#gameScreen` wrapper divs were already created in Task 1. Implement the toggle functions:

```javascript
function showMenu() {
  menuRecord.textContent = highScore.toLocaleString('en-US');
  menuScreen.style.display = 'flex';
  gameScreen.style.display = 'none';
  overlay.classList.remove('visible');
  // Clear board
  board.querySelectorAll('.snake-segment, .food').forEach(el => el.remove());
}

function showGameplay() {
  menuScreen.style.display = 'none';
  gameScreen.style.display = 'flex';
}
```

Update `startGame()` to call `showGameplay()` before rendering.

- [ ] **Step 6: Implement overlay show/hide for pause and game over**

```javascript
function showOverlay(type, isNewRecord) {
  if (type === 'pause') {
    overlayTitle.textContent = 'PAUSED';
    overlaySubtitle.textContent = 'TAP TO RESUME';
    overlayScore.style.display = 'none';
    overlayRecord.style.display = 'none';
    overlayBtnPrimary.textContent = 'RESUME';
    overlayBtnSecondary.textContent = 'QUIT';
    overlayBtnPrimary.onclick = resumeGame;
    overlayBtnSecondary.onclick = returnToMenu;
  } else {
    overlayTitle.textContent = 'GAME OVER';
    overlaySubtitle.textContent = '';
    overlayScore.textContent = score.toLocaleString('en-US');
    overlayScore.style.display = 'block';
    overlayRecord.style.display = isNewRecord ? 'block' : 'none';
    overlayRecord.textContent = 'NEW RECORD';
    if (isNewRecord) overlayRecord.classList.add('pulse');
    overlayBtnPrimary.textContent = 'PLAY AGAIN';
    overlayBtnSecondary.textContent = 'MENU';
    overlayBtnPrimary.onclick = startGame;
    overlayBtnSecondary.onclick = returnToMenu;
  }
  overlay.classList.add('visible');
}

function hideOverlay() {
  overlay.classList.remove('visible');
  overlayRecord.classList.remove('pulse');
}
```

- [ ] **Step 7: Implement level up visual feedback**

```javascript
function onLevelUp() {
  // Flash rank bar
  rankFill.style.opacity = '1';
  rankFill.style.boxShadow = '0 0 16px rgba(212, 169, 76, 0.6)';
  setTimeout(() => {
    rankFill.style.opacity = '';
    rankFill.style.boxShadow = '';
  }, 400);

  // Scale rank numeral
  rankTitle.style.transform = 'scale(1.3)';
  setTimeout(() => { rankTitle.style.transform = 'scale(1)'; }, 300);

  playSound('levelup');
}
```

- [ ] **Step 8: Verify rendering works end to end**

Open in browser, click Play. Verify:
- Snake renders as gold segments on the board
- Food renders as pulsing ruby gem
- Snake moves, grows on eating food
- Score/chain/rank update correctly
- Pause overlay shows and hides
- Game over overlay shows with correct score
- New record detection works
- Menu ↔ gameplay transitions work

- [ ] **Step 9: Commit**

```bash
git add aureum-snake.html
git commit -m "feat: connect game engine to DOM — snake rendering, UI updates, screen transitions"
```

---

### Task 4: Input — D-pad and Keyboard Controls

Wire the D-pad buttons and keyboard arrow keys to the game engine direction changes. Add pause via spacebar and start via Enter.

**Files:**
- Modify: `aureum-snake.html`

- [ ] **Step 1: Wire D-pad button click/touch events**

The `data-dir` attributes were already added to buttons in Task 1 Step 6.

```javascript
const dirMap = {
  up: { x: 0, y: -1 },
  down: { x: 0, y: 1 },
  left: { x: -1, y: 0 },
  right: { x: 1, y: 0 }
};

document.querySelectorAll('.ctrl-btn[data-dir]').forEach(btn => {
  function handlePress(e) {
    e.preventDefault();
    const dir = dirMap[btn.dataset.dir];
    if (dir && gameState === 'playing') {
      setDirection(dir.x, dir.y);
      playSound('click');
      vibrate(10);
    }
  }
  btn.addEventListener('touchstart', handlePress, { passive: false });
  btn.addEventListener('mousedown', handlePress);
});
```

- [ ] **Step 2: Wire keyboard controls**

```javascript
document.addEventListener('keydown', (e) => {
  const key = e.key;
  if (key === 'ArrowUp' || key === 'ArrowDown' || key === 'ArrowLeft' || key === 'ArrowRight') {
    e.preventDefault();
    const keyDir = {
      ArrowUp: { x: 0, y: -1 },
      ArrowDown: { x: 0, y: 1 },
      ArrowLeft: { x: -1, y: 0 },
      ArrowRight: { x: 1, y: 0 }
    };
    if (gameState === 'playing') {
      setDirection(keyDir[key].x, keyDir[key].y);
    }
  } else if (key === ' ') {
    e.preventDefault();
    if (gameState === 'playing') pauseGame();
    else if (gameState === 'paused') resumeGame();
  } else if (key === 'Enter') {
    if (gameState === 'menu' || gameState === 'gameover') startGame();
  }
});
```

- [ ] **Step 3: Wire Play, Pause, and overlay button clicks**

```javascript
document.getElementById('playBtn').addEventListener('click', startGame);
document.getElementById('pauseBtn').addEventListener('click', () => {
  if (gameState === 'playing') pauseGame();
});

// Menu play button
document.getElementById('menuPlayBtn').addEventListener('click', startGame);
```

- [ ] **Step 4: Add visibility change auto-pause**

```javascript
document.addEventListener('visibilitychange', () => {
  if (document.hidden && gameState === 'playing') {
    pauseGame();
  }
});
```

- [ ] **Step 5: Verify all inputs work**

Test in browser:
- D-pad buttons change snake direction on touch/click
- Arrow keys work on desktop
- Spacebar pauses/resumes
- Enter starts game from menu and game over
- Tab switching auto-pauses
- 180-degree reversal is prevented

- [ ] **Step 6: Commit**

```bash
git add aureum-snake.html
git commit -m "feat: add D-pad touch controls, keyboard input, and auto-pause"
```

---

### Task 5: Audio — Web Audio API Sound Synthesis

Add synthesized sound effects for eat, level up, game over, and button click. Add speaker toggle to mute/unmute with localStorage persistence.

**Files:**
- Modify: `aureum-snake.html`

- [ ] **Step 1: Add AudioContext and sound toggle state**

```javascript
let audioCtx = null;
let soundEnabled = localStorage.getItem('aureum-snake-sound') !== 'off';

function ensureAudioCtx() {
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  }
  if (audioCtx.state === 'suspended') {
    audioCtx.resume();
  }
}
```

- [ ] **Step 2: Implement sound effect functions**

```javascript
function playSound(type) {
  if (!soundEnabled) return;
  ensureAudioCtx();

  if (type === 'eat') {
    // Bright chime — sine sweep 600→900Hz, 80ms
    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();
    osc.type = 'sine';
    osc.frequency.setValueAtTime(600, audioCtx.currentTime);
    osc.frequency.linearRampToValueAtTime(900, audioCtx.currentTime + 0.08);
    gain.gain.setValueAtTime(0.15, audioCtx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.12);
    osc.connect(gain).connect(audioCtx.destination);
    osc.start();
    osc.stop(audioCtx.currentTime + 0.12);
  } else if (type === 'levelup') {
    // Ascending arpeggio — C5 E5 G5, 60ms each, 30ms gap
    const notes = [523, 659, 784];
    notes.forEach((freq, i) => {
      const osc = audioCtx.createOscillator();
      const gain = audioCtx.createGain();
      const start = audioCtx.currentTime + i * 0.09;
      osc.type = 'sine';
      osc.frequency.setValueAtTime(freq, start);
      gain.gain.setValueAtTime(0.12, start);
      gain.gain.exponentialRampToValueAtTime(0.001, start + 0.08);
      osc.connect(gain).connect(audioCtx.destination);
      osc.start(start);
      osc.stop(start + 0.08);
    });
  } else if (type === 'gameover') {
    // Descending tone — triangle 400→150Hz, 400ms (spec mentions "slight reverb" but
    // omitted here for simplicity in single-file context — the long decay tail provides
    // a similar effect)
    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();
    osc.type = 'triangle';
    osc.frequency.setValueAtTime(400, audioCtx.currentTime);
    osc.frequency.linearRampToValueAtTime(150, audioCtx.currentTime + 0.4);
    gain.gain.setValueAtTime(0.15, audioCtx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.5);
    osc.connect(gain).connect(audioCtx.destination);
    osc.start();
    osc.stop(audioCtx.currentTime + 0.5);
  } else if (type === 'click') {
    // Noise burst — 15ms, low volume
    const bufferSize = audioCtx.sampleRate * 0.015;
    const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate);
    const data = buffer.getChannelData(0);
    for (let i = 0; i < bufferSize; i++) {
      data[i] = (Math.random() * 2 - 1) * 0.08;
    }
    const source = audioCtx.createBufferSource();
    source.buffer = buffer;
    source.connect(audioCtx.destination);
    source.start();
  }
}
```

- [ ] **Step 3: Implement vibrate helper**

```javascript
function vibrate(pattern) {
  if (soundEnabled && navigator.vibrate) {
    navigator.vibrate(pattern);
  }
}
```

- [ ] **Step 4: Wire sound toggle button**

```javascript
function updateSoundIcon() {
  soundBtn.style.opacity = soundEnabled ? '1' : '0.3';
}

soundBtn.addEventListener('click', () => {
  soundEnabled = !soundEnabled;
  localStorage.setItem('aureum-snake-sound', soundEnabled ? 'on' : 'off');
  updateSoundIcon();
  ensureAudioCtx(); // Initialize on first interaction
});

updateSoundIcon();
```

- [ ] **Step 5: Add game over sound and haptic calls**

Update the `gameOver()` function to include:
```javascript
playSound('gameover');
vibrate([50, 30, 100]);
```

- [ ] **Step 6: Verify audio works**

Test in browser:
- Eating food plays chime
- Level up plays arpeggio
- Game over plays descending tone
- D-pad clicks play subtle click
- Speaker icon toggles sound on/off
- Sound preference persists across page reload
- Haptics fire on mobile (test with device or emulator)

- [ ] **Step 7: Commit**

```bash
git add aureum-snake.html
git commit -m "feat: add synthesized sound effects, haptic feedback, and sound toggle"
```

---

### Task 6: Polish — Final Tuning and Edge Cases

Fix any remaining issues, add the snake/food CSS from the existing mockup, ensure all animations are smooth, handle edge cases.

**Files:**
- Modify: `aureum-snake.html`

- [ ] **Step 1: Add snake segment and food CSS**

Ensure these CSS rules exist (from spec Section 2.2):
```css
.snake-segment { position: absolute; width: var(--cell); height: var(--cell); display: flex; align-items: center; justify-content: center; }
.gem { border-radius: 3px; transition: all 0.1s ease; }
.snake-segment.head .gem { width: calc(var(--cell) - 2px); height: calc(var(--cell) - 2px); border-radius: 4px; background: linear-gradient(135deg, var(--gold-light), var(--gold), var(--gold-dark)); box-shadow: 0 0 8px rgba(212,169,76,0.4), inset 0 1px 0 rgba(255,255,255,0.15); }
.snake-segment.body .gem { width: calc(var(--cell) - 4px); height: calc(var(--cell) - 4px); background: linear-gradient(135deg, var(--gold), var(--gold-dark)); box-shadow: 0 0 4px rgba(212,169,76,0.2); }
.snake-segment.tail .gem { width: calc(var(--cell) - 7px); height: calc(var(--cell) - 7px); border-radius: 50%; background: var(--gold-dark); opacity: 0.4; }
.food { position: absolute; width: var(--cell); height: var(--cell); display: flex; align-items: center; justify-content: center; }
.ruby { width: calc(var(--cell) - 4px); height: calc(var(--cell) - 4px); background: linear-gradient(135deg, var(--ruby), var(--burgundy)); border-radius: 50%; box-shadow: 0 0 10px rgba(212,68,68,0.4), 0 0 25px rgba(212,68,68,0.15), inset 0 1px 0 rgba(255,255,255,0.2); animation: rubyGlow 2.5s ease-in-out infinite; }
@keyframes rubyGlow { 0%,100% { transform: scale(1); box-shadow: 0 0 10px rgba(212,68,68,0.4), 0 0 25px rgba(212,68,68,0.15), inset 0 1px 0 rgba(255,255,255,0.2); } 50% { transform: scale(1.08); box-shadow: 0 0 14px rgba(212,68,68,0.5), 0 0 35px rgba(212,68,68,0.2), inset 0 1px 0 rgba(255,255,255,0.25); } }
```

- [ ] **Step 2: Verify direction queue handles rapid inputs correctly**

The direction queue was already implemented in Task 2. Test: rapidly press right then down before a tick fires. The snake should turn right then down on consecutive ticks, not skip the first input.

- [ ] **Step 3: Add transition for score value element**

```css
.score-value { transition: transform 0.15s ease-out; }
.rank-fill { transition: width 0.3s ease-out, opacity 0.3s, box-shadow 0.3s; }
.rank-title { transition: transform 0.3s ease-out; }
```

- [ ] **Step 4: Ensure menu shows on initial page load**

```javascript
// At end of script:
showMenu();
```

- [ ] **Step 5: Full end-to-end playtest**

Play through the complete game flow:
- Open page → start screen with animations
- Click Play → gameplay starts, snake moves
- Eat food → score increases, chime plays, chain updates
- Eat 5 food → level up to II, arpeggio plays, rank bar resets
- Hit wall → game over, descending tone, correct score shown
- Beat high score → "NEW RECORD" appears with pulse
- Play Again → restarts correctly
- Menu → returns to start screen with updated record
- Pause → overlay shows, resume works
- Tab away → auto-pauses
- Sound toggle → persists, mutes all sound + haptics
- Keyboard → arrows, space, enter all work
- Responsive → test at various viewport sizes

- [ ] **Step 6: Commit**

```bash
git add aureum-snake.html
git commit -m "feat: polish — snake/food CSS, direction queue, transitions, edge cases"
```

---

## Summary

| Task | What | Commit Message |
|------|------|----------------|
| 1 | HTML/CSS layout — Art Deco styling | `feat: AUREUM snake game — static HTML/CSS layout with Art Deco styling` |
| 2 | Game engine — state, movement, collision, scoring | `feat: add core snake game engine` |
| 3 | DOM rendering — connect state to UI | `feat: connect game engine to DOM` |
| 4 | Input — D-pad + keyboard + auto-pause | `feat: add D-pad touch controls, keyboard input, and auto-pause` |
| 5 | Audio — Web Audio synthesis + haptics | `feat: add synthesized sound effects, haptic feedback, and sound toggle` |
| 6 | Polish — CSS, direction queue, edge cases | `feat: polish — snake/food CSS, direction queue, transitions, edge cases` |
