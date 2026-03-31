# AUREUM — Premium Mobile Snake Game

**Date:** 2026-03-30
**Format:** Single self-contained HTML file
**Platform:** Mobile-first (responsive, works on desktop)
**Aesthetic:** Art Deco luxury — gold on obsidian

---

## 1. Overview

A fully playable, polished snake game in a single HTML file. Art Deco visual identity with gold/obsidian palette, Playfair Display + Outfit typography, shimmer borders, and premium micro-interactions. Mobile-first with on-screen D-pad controls (swipe gestures intentionally excluded — D-pad is the sole touch input). No external dependencies beyond Google Fonts.

### 1.1 Color Palette

```css
--gold: #d4a94c;
--gold-light: #e8c878;
--gold-dark: #a07830;
--cream: #f8f4eb;
--obsidian: #0e0e12;
--charcoal: #1a1a22;
--surface: #141418;
--burgundy: #7a2332;
--ruby: #d44444;
--text-light: rgba(248, 244, 235, 0.85);
--text-dim: rgba(248, 244, 235, 0.3);
```

## 2. Screens & Flow

```
Start Screen → Gameplay → Pause (overlay) → Game Over (overlay) → Start Screen
                  ↑                                    |
                  +-------- Play Again ----------------+
```

### 2.1 Start Screen

- AUREUM title in Playfair Display 900, gold, letter-spacing 10px
- Diamond ornament line above title
- "Serpentis Ludus" subtitle below
- Empty game board visible behind (decorative grid with corner marks, gold shimmer border)
- High score display: "Record: 8,120" in gold, centered below board
- Prominent "PLAY" button — gold gradient, Outfit font, uppercase, letter-spacing 4px
- Speaker icon (top-right) — toggles sound/haptics on/off (100% opacity = on, 30% = off)
- Staggered fade-in entrance animations (header 0.2s, board 0.6s, button 1.0s)

### 2.2 Gameplay Screen

**Header area:**
- Score display bar with 3 sections: Score (featured, gold), Record, Chain (snake length)
- Gold corner decorations on score container
- Dividers between sections at 20% inset

**Game board:**
- 18x18 grid
- 3px gold shimmer border (animated gradient: gold → gold-dark → gold, 6s loop)
- Inner 8px obsidian padding
- Subtle gold grid lines (3% opacity)
- Corner decorations inside board (20px, 8% opacity)

**Snake rendering:**
- Head: `cellSize - 2px` wide/tall (16px on mobile, 20px on desktop), gold gradient (light → mid → dark), 4px border-radius, gold glow shadow
- Body: `cellSize - 4px` wide/tall (14px on mobile, 18px on desktop), gold gradient, opacity fades from 1.0 to 0.5 head-to-tail
- Tail (last segment): `cellSize - 7px`, circular, 40% opacity

**Food rendering:**
- Ruby gem: circular, red-to-burgundy gradient
- Pulsing glow animation (2.5s ease-in-out infinite)
- Red box-shadow glow

**Rank bar (below board):**
- "Rank" label in Playfair Display italic
- Track: 2px, gold 8% opacity background
- Fill: gold gradient with glow, width = progress toward next level
- Rank number: Roman numerals (I through X) in Playfair Display bold

**D-pad controls:**
- 2-row compact layout: Row 1: [spacer][up][spacer], Row 2: [left][down][right]
- 62x62px buttons, charcoal background, gold 10% border
- Inner decorative border (3px inset, gold 4% opacity)
- Arrow icons via inline SVG, 18px, stroke-width 1.5
- Active state: gold 8% background, gold 25% border, scale 0.93, gold icon color

**Bottom bar:**
- Pause button (ghost style): gold 15% border, dim text
- (Play button hidden during gameplay)

### 2.3 Pause Overlay

- Covers game board only (not full screen)
- Background: obsidian at 85% opacity with backdrop-filter blur(8px)
- "PAUSED" title: Playfair Display 900, gold gradient text, letter-spacing 8px
- "TAP TO RESUME" subtitle: Outfit, dim text, letter-spacing 3px
- Two buttons: "Resume" (gold gradient) and "Quit" (ghost)
- Triggered by pause button or `document.visibilitychange` when `hidden` (e.g., switching tabs/apps)

### 2.4 Game Over Overlay

- Same overlay style as pause (covers game board)
- "GAME OVER" title in gold
- Final score displayed large: Playfair Display 700, gold
- If new record: "NEW RECORD" label with brief gold pulse animation
- Two buttons: "Play Again" (gold gradient) and "Menu" (ghost)

## 3. Game Mechanics

### 3.1 Grid & Movement

- 18x18 cell grid (0-indexed from top-left, x=column, y=row)
- Snake starts at length 3, moving right: head at (9, 9), body at (8, 9) and (7, 9)
- One cell per tick movement
- Direction change queued (prevents 180-degree reversals)
- Wall collision = game over
- Self collision = game over

### 3.2 Food & Scoring

- One food item on board at all times
- Food spawns at random empty cell
- Eating food: snake grows by 1, score increases
- Score formula: `10 * currentLevel`
- "Chain" display = current snake length
- Scores displayed with comma thousands separator (`toLocaleString('en-US')`)

### 3.3 Levels & Speed

- Level starts at I (1)
- Every 5 food items eaten = level up (max level X / 10)
- Tick interval: `150 - (level - 1) * 8` ms (min 78ms at level X)
- On level up: rank bar fills, Roman numeral updates, brief flash effect

### 3.4 High Score

- Stored in `localStorage` under key `aureum-snake-highscore`
- Checked on game over
- Displayed on start screen and in gameplay stats bar

## 4. Audio (Web Audio API)

All sounds synthesized — no external audio files.

### 4.1 Sound Effects

| Event | Sound | Parameters |
|-------|-------|------------|
| Eat food | Bright chime | Short sine oscillator, frequency sweep 600→900Hz, 80ms duration |
| Level up | Ascending arpeggio | 3-note sequence (C5→E5→G5), sine, 60ms each, 30ms gap |
| Game over | Descending tone | Triangle wave, 400→150Hz sweep, 400ms, slight reverb |
| Button press | Subtle click | Noise burst, 15ms, low volume |

### 4.2 Sound Toggle

- Speaker icon in header toggles sound and haptics on/off
- State persisted to `localStorage` under `aureum-snake-sound`
- Icon changes opacity to indicate state (100% on, 30% off)

## 5. Haptic Feedback

- `navigator.vibrate(30)` on food eat
- `navigator.vibrate([50, 30, 100])` on game over
- `navigator.vibrate(10)` on D-pad button press
- Only fires if `navigator.vibrate` exists (mobile only)
- Respects sound toggle (sound off = haptics off). Intentionally coupled to keep UI simple — no separate haptic toggle.

## 6. Visual Effects & Animations

### 6.1 Entry Animations

Staggered `decoReveal` keyframe (translateY 8px → 0, opacity 0 → 1):
- Header: 0.2s delay
- Score bar: 0.4s delay
- Game board: 0.6s delay
- Rank bar: 0.75s delay
- Controls: 0.85s delay
- Bottom bar: 1.0s delay

### 6.2 Gameplay Animations

- **Gold shimmer border:** `background-position` animation on game frame, 6s ease-in-out infinite
- **Food pulse:** Scale 1→1.08 and shadow intensity cycle, 2.5s infinite
- **Score change:** Brief `transform: scale(1.15)` on score value, 150ms ease-out
- **Level up:** Rank bar fill flashes brighter (opacity pulse), rank numeral scales up briefly
- **New record:** Gold pulse ring animation on game over overlay

### 6.3 Background

- Radial gold gradients (6% opacity) at top and bottom of viewport
- Vertical gold accent lines at top/bottom center (1px, 60px, 20% opacity)
- Side accent lines at viewport midpoint (1px, 200px, 12% opacity)

## 7. Responsive Design

### 7.1 Mobile (default)

- Max-width: 420px, centered
- Cell size: 18px (board = 324px)
- Controls and stats sized for thumb reach
- `env(safe-area-inset-bottom)` padding for notched devices
- `-webkit-tap-highlight-color: transparent`
- `user-select: none` on body

### 7.2 Larger Screens (>420px width)

- Cell size increases to 22px (board = 396px)
- Layout remains single-column centered
- Side accent lines visible

### 7.3 Short Screens (<700px height)

- Reduced padding on score panel and controls
- Smaller stat values and control buttons
- Compressed rank bar margins

## 8. Keyboard Support (Desktop)

- Arrow keys map to D-pad directions
- Space bar toggles pause
- Enter key starts game / restarts after game over
- Keyboard does not conflict with D-pad (both work simultaneously)

## 9. Technical Notes

- Single HTML file, inline CSS and JS
- Google Fonts loaded via `<link>`: Playfair Display (400, 700, 900, italic 400) + Outfit (200, 300, 400, 500)
- Game loop via `requestAnimationFrame` with tick accumulator (not `setInterval`)
- Canvas-free: only snake segments and food are rendered as absolutely positioned `<div>` elements within the game board container (not all 324 cells). Grid lines via CSS `background-image` on the board.
- Web Audio API `AudioContext` created on first user interaction (avoids autoplay policy)
- No framework dependencies
