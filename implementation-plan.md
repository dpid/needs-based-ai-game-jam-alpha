# Implementation Plan: Gravity Flip

## Overview

A single-file HTML5 Canvas game where the player controls a square that flips gravity on input, navigating through procedurally generated obstacles. The entire game (HTML, CSS, JavaScript) fits in one `index.html` file under 200 lines.

**Technical Approach:** Pure vanilla JavaScript with Canvas 2D API, focusing on minimal abstractions and inline game state management. No functions for simple operations - prioritize code density over modularity.

## Tech Stack

- **Platform:** Single HTML file (index.html)
- **Rendering:** Canvas 2D Context API
- **Game Loop:** requestAnimationFrame
- **Audio:** Web Audio API (AudioContext + OscillatorNode)
- **Storage:** localStorage for high score
- **Language:** JavaScript (ES6+)
- **No external dependencies:** Zero npm packages, libraries, or asset files

## Architecture Decisions

### 1. Single-File Structure
All code lives in one HTML file with three sections:
- `<canvas>` element in body
- `<style>` block for basic canvas styling
- `<script>` block containing all game logic

### 2. Flat State Management
No classes or complex objects. Global variables for:
- Player state (x, y, velocity, size)
- Game state (score, gameOver, highScore)
- Obstacle array (simple objects)
- Constants (gravity, speeds, dimensions)

### 3. Inline Game Loop
Single `update()` function called by requestAnimationFrame handles:
- Input processing (flag-based)
- Physics updates
- Obstacle management
- Collision detection
- Rendering
- All in sequence, no separation of concerns

### 4. Procedural Obstacle Generation
Obstacles spawn at fixed intervals. Each obstacle is:
```javascript
{ x: canvasWidth, gapY: number, gapHeight: number }
```
Gap position alternates top/bottom. Gap height decreases with score.

### 5. Audio Implementation Strategy
Create AudioContext once globally. On flip, create short-lived OscillatorNode with frequency based on gravity direction. Keep implementation under 10 lines total.

## Project Structure

```
game-repo/
└── index.html    (Complete game, ~190 lines)
```

## Code Organization Within index.html

```html
<!DOCTYPE html>
<html>
<head>
  <title>Gravity Flip</title>
  <style>
    /* 5 lines: full-screen canvas centering */
  </style>
</head>
<body>
  <canvas id="c"></canvas>
  <script>
    // Section 1: Setup & Constants (15 lines)
    // Section 2: Game State Variables (10 lines)
    // Section 3: Input Handlers (8 lines)
    // Section 4: Game Loop - Update Function (120 lines)
    // Section 5: Initialization (5 lines)
  </script>
</body>
</html>
```

## Constants and Configuration

```javascript
const CANVAS_WIDTH = 800;
const CANVAS_HEIGHT = 600;
const PLAYER_SIZE = 20;
const PLAYER_X = 200;
const GRAVITY = 0.5;
const TERMINAL_VELOCITY = 8;
const FLIP_VELOCITY = -10;
const OBSTACLE_WIDTH = 40;
const OBSTACLE_SPEED = 3;
const OBSTACLE_SPAWN_INTERVAL = 180;
const GAP_BASE = 150;
const GAP_MIN = 75;
const SCORE_INTERVAL = 100;
```

## State Variables

```javascript
let canvas, ctx, audioCtx;
let playerY, playerVY, playerRotation;
let obstacles = [];
let score, highScore, gameOver;
let flipInput, lastGapTop;
let frameCount, scoreTimer;
```

## Implementation Phases

### Phase 1: Canvas Setup & Basic Structure
**Goal:** Black screen with canvas ready for rendering
**Estimated Lines:** 25

**Steps:**
1. Create HTML structure with canvas element
2. Add CSS for full-screen centered canvas with black background
3. Get canvas context and set dimensions
4. Initialize AudioContext (with user gesture handling)
5. Set up basic requestAnimationFrame loop

**Code Structure:**
```html
<style>
  body { margin: 0; background: #000; display: flex; justify-content: center; align-items: center; height: 100vh; }
  canvas { border: 1px solid #333; }
</style>
<canvas id="c"></canvas>
<script>
  const canvas = document.getElementById('c');
  const ctx = canvas.getContext('2d');
  canvas.width = 800;
  canvas.height = 600;
  let audioCtx;

  function init() {
    audioCtx = new AudioContext();
    // ... rest of initialization
  }

  function update() {
    requestAnimationFrame(update);
  }

  document.addEventListener('keydown', init, { once: true });
  document.addEventListener('click', init, { once: true });
</script>
```

**Verification:**
- Canvas appears centered on black background
- Console shows no errors
- Frame loop runs (add console.log to verify)

### Phase 2: Player Rendering & Physics
**Goal:** Cyan square falls with gravity, bounces at floor/ceiling
**Estimated Lines:** +30 (Total: 55)

**Steps:**
1. Initialize player state variables (y, vy, rotation)
2. Add gravity physics in update loop
3. Clamp player to canvas bounds
4. Render player as filled rectangle with rotation
5. Add visual feedback with rotation based on velocity

**Code to Add:**
```javascript
// State initialization
let playerY = 300;
let playerVY = 0;
let playerRotation = 0;

// In update function
playerVY += GRAVITY;
if (Math.abs(playerVY) > TERMINAL_VELOCITY) {
  playerVY = playerVY > 0 ? TERMINAL_VELOCITY : -TERMINAL_VELOCITY;
}
playerY += playerVY;

if (playerY > canvas.height - PLAYER_SIZE) {
  playerY = canvas.height - PLAYER_SIZE;
  playerVY = 0;
}
if (playerY < 0) {
  playerY = 0;
  playerVY = 0;
}

playerRotation = playerVY * 2;

// Rendering
ctx.fillStyle = '#000';
ctx.fillRect(0, 0, canvas.width, canvas.height);
ctx.save();
ctx.translate(PLAYER_X + PLAYER_SIZE / 2, playerY + PLAYER_SIZE / 2);
ctx.rotate(playerRotation * Math.PI / 180);
ctx.fillStyle = '#0FF';
ctx.fillRect(-PLAYER_SIZE / 2, -PLAYER_SIZE / 2, PLAYER_SIZE, PLAYER_SIZE);
ctx.restore();
```

**Verification:**
- Square starts at center and falls
- Square stops at bottom edge
- Square rotates as it falls
- Cyan color (#00FFFF) visible against black

### Phase 3: Input & Gravity Flip
**Goal:** Any key or click flips gravity direction
**Estimated Lines:** +15 (Total: 70)

**Steps:**
1. Add keydown and click event listeners
2. Set flip input flag on any input
3. In update loop, check flag and reverse gravity
4. Reset flag after processing
5. Add audio feedback (oscillator beep)

**Code to Add:**
```javascript
let flipInput = false;
let gravityDirection = 1;

document.addEventListener('keydown', () => {
  if (audioCtx.state === 'suspended') audioCtx.resume();
  flipInput = true;
});

document.addEventListener('click', () => {
  if (audioCtx.state === 'suspended') audioCtx.resume();
  flipInput = true;
});

// In update loop
if (flipInput) {
  gravityDirection *= -1;
  playerVY = FLIP_VELOCITY * gravityDirection;

  const osc = audioCtx.createOscillator();
  const gain = audioCtx.createGain();
  osc.connect(gain);
  gain.connect(audioCtx.destination);
  osc.frequency.value = gravityDirection < 0 ? 440 : 220;
  gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.1);
  osc.start();
  osc.stop(audioCtx.currentTime + 0.1);

  flipInput = false;
}

playerVY += GRAVITY * gravityDirection;
```

**Verification:**
- Pressing any key reverses gravity
- Clicking reverses gravity
- Audio beep plays (high pitch up, low pitch down)
- Square changes direction immediately

### Phase 4: Obstacle Generation & Scrolling
**Goal:** White barriers scroll from right with gaps
**Estimated Lines:** +35 (Total: 105)

**Steps:**
1. Create obstacles array
2. Add spawn logic based on frame counter
3. Alternate gap positions (top/bottom)
4. Update obstacle positions (scroll left)
5. Remove off-screen obstacles
6. Render obstacles as white rectangles

**Code to Add:**
```javascript
let obstacles = [];
let frameCount = 0;
let lastGapTop = false;

// In update loop
frameCount++;

if (frameCount % OBSTACLE_SPAWN_INTERVAL === 0) {
  const gapHeight = Math.max(GAP_BASE - score * 0.05, GAP_MIN);
  const gapY = lastGapTop ? canvas.height - gapHeight - 50 : 50;
  obstacles.push({ x: canvas.width, gapY, gapHeight });
  lastGapTop = !lastGapTop;
}

obstacles.forEach(obs => {
  obs.x -= OBSTACLE_SPEED;
});

obstacles = obstacles.filter(obs => obs.x > -OBSTACLE_WIDTH);

// Rendering obstacles
ctx.fillStyle = '#FFF';
obstacles.forEach(obs => {
  ctx.fillRect(obs.x, 0, OBSTACLE_WIDTH, obs.gapY);
  ctx.fillRect(obs.x, obs.gapY + obs.gapHeight, OBSTACLE_WIDTH, canvas.height);
});
```

**Verification:**
- White barriers appear from right edge
- Gaps alternate between top and bottom
- Obstacles scroll smoothly at constant speed
- Old obstacles disappear off left edge

### Phase 5: Collision Detection & Game Over
**Goal:** Collision ends game, shows restart prompt
**Estimated Lines:** +25 (Total: 130)

**Steps:**
1. Add gameOver state variable
2. Implement AABB collision detection
3. Stop game loop updates when gameOver is true
4. Display "GAME OVER" and score
5. Reset game on input after game over

**Code to Add:**
```javascript
let gameOver = false;

// In update loop (before physics)
if (gameOver) {
  ctx.fillStyle = '#FFF';
  ctx.font = '48px Arial';
  ctx.textAlign = 'center';
  ctx.fillText('GAME OVER', canvas.width / 2, canvas.height / 2);
  ctx.font = '24px Arial';
  ctx.fillText('Score: ' + score, canvas.width / 2, canvas.height / 2 + 40);
  ctx.fillText('Press any key to restart', canvas.width / 2, canvas.height / 2 + 80);
  return;
}

// Collision detection
obstacles.forEach(obs => {
  if (PLAYER_X + PLAYER_SIZE > obs.x && PLAYER_X < obs.x + OBSTACLE_WIDTH) {
    if (playerY < obs.gapY || playerY + PLAYER_SIZE > obs.gapY + obs.gapHeight) {
      gameOver = true;

      const osc = audioCtx.createOscillator();
      const gain = audioCtx.createGain();
      osc.connect(gain);
      gain.connect(audioCtx.destination);
      osc.type = 'sawtooth';
      osc.frequency.value = 80;
      gain.gain.setValueAtTime(0.2, audioCtx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.3);
      osc.start();
      osc.stop(audioCtx.currentTime + 0.3);
    }
  }
});

// In input handlers
if (gameOver) {
  playerY = 300;
  playerVY = 0;
  gravityDirection = 1;
  obstacles = [];
  score = 0;
  frameCount = 0;
  gameOver = false;
  return;
}
```

**Verification:**
- Hitting obstacle triggers game over
- Game stops updating
- "GAME OVER" message displays
- Collision sound plays
- Any key restarts game cleanly

### Phase 6: Score System & Difficulty Scaling
**Goal:** Score increases over time, gaps narrow with score
**Estimated Lines:** +20 (Total: 150)

**Steps:**
1. Add score and scoreTimer variables
2. Increment score every 100ms
3. Display score in top-left corner
4. Modify gap height based on score
5. Add high score with localStorage

**Code to Add:**
```javascript
let score = 0;
let scoreTimer = 0;
let highScore = parseInt(localStorage.getItem('highScore') || '0');

// In update loop
scoreTimer++;
if (scoreTimer >= SCORE_INTERVAL / 16.67) {
  score++;
  scoreTimer = 0;
}

// When spawning obstacles (modify existing code)
const gapHeight = Math.max(GAP_BASE - score * 0.05, GAP_MIN);

// Rendering score
ctx.fillStyle = '#FFF';
ctx.font = '24px Arial';
ctx.textAlign = 'left';
ctx.fillText('Score: ' + score, 20, 40);
ctx.fillText('Best: ' + highScore, 20, 70);

// On game over
if (score > highScore) {
  highScore = score;
  localStorage.setItem('highScore', highScore);
}
```

**Verification:**
- Score increments continuously during play
- Score displays in top-left
- Gaps get narrower as score increases
- High score persists after page reload

### Phase 7: Visual Polish & Final Touches
**Goal:** Complete the minimalist aesthetic
**Estimated Lines:** +40 (Total: 190)

**Steps:**
1. Add boundary lines at top/bottom
2. Add intro text overlay on first load
3. Improve text rendering (shadows, better fonts)
4. Add subtle glow effect to player
5. Fine-tune colors and spacing

**Code to Add:**
```javascript
let firstPlay = true;

// Boundary lines rendering
ctx.strokeStyle = '#FFF';
ctx.lineWidth = 2;
ctx.beginPath();
ctx.moveTo(0, 0);
ctx.lineTo(canvas.width, 0);
ctx.moveTo(0, canvas.height);
ctx.lineTo(canvas.width, canvas.height);
ctx.stroke();

// Intro overlay
if (firstPlay && frameCount < 300) {
  ctx.fillStyle = 'rgba(255, 255, 255, 0.8)';
  ctx.font = '32px Arial';
  ctx.textAlign = 'center';
  ctx.fillText('PRESS ANY KEY TO FLIP', canvas.width / 2, canvas.height / 2);
}

// Player glow effect
ctx.shadowBlur = 15;
ctx.shadowColor = '#0FF';
ctx.fillStyle = '#0FF';
ctx.fillRect(-PLAYER_SIZE / 2, -PLAYER_SIZE / 2, PLAYER_SIZE, PLAYER_SIZE);
ctx.shadowBlur = 0;

// In input handler
firstPlay = false;
```

**Verification:**
- Intro text appears on first load
- Intro text fades after 5 seconds
- Player has cyan glow effect
- Boundary lines visible at edges
- Overall aesthetic matches GDD vision

## Function Signatures & Responsibilities

Given the 200 LOC constraint, functions are minimal. The entire game uses:

### Core Functions

```javascript
function init()
```
- Initialize AudioContext on first user interaction
- Called once via event listener
- Lines: 3-5

```javascript
function update()
```
- Main game loop (called by requestAnimationFrame)
- Handles all game logic inline:
  - Input processing
  - Physics updates
  - Obstacle management
  - Collision detection
  - Rendering
- Lines: 120-140

### No Helper Functions
To stay under 200 LOC, avoid helper functions. Inline all logic in the update loop with clear variable names.

## State Management Approach

**Global Scope Variables:**
```javascript
const canvas, ctx, audioCtx;
let playerY, playerVY, playerRotation;
let gravityDirection = 1;
let obstacles = [];
let score = 0, highScore = 0, gameOver = false;
let flipInput = false, firstPlay = true;
let frameCount = 0, scoreTimer = 0, lastGapTop = false;
```

**State Transitions:**
- INTRO (firstPlay = true) → PLAYING (firstPlay = false)
- PLAYING (gameOver = false) → GAME_OVER (gameOver = true)
- GAME_OVER → PLAYING (reset all state variables)

**No State Machine:** Too verbose for LOC constraint. Use boolean flags.

## Game Loop Structure

```javascript
function update() {
  requestAnimationFrame(update);

  // 1. Handle game over state
  if (gameOver) {
    // Render game over screen
    return;
  }

  // 2. Process input
  if (flipInput) {
    // Flip gravity, play sound
    flipInput = false;
  }

  // 3. Update physics
  // Apply gravity, update velocity, update position, clamp bounds

  // 4. Update score
  // Increment scoreTimer, add to score

  // 5. Spawn obstacles
  // Check frameCount, create new obstacle

  // 6. Update obstacles
  // Move left, remove off-screen

  // 7. Collision detection
  // Check player vs obstacles, set gameOver

  // 8. Render
  // Clear screen, draw boundaries, draw obstacles, draw player, draw UI
}
```

## Testing & Verification Strategy

### Manual Testing Checklist

**Phase 1-2: Basic Mechanics**
- [ ] Canvas renders at 800x600
- [ ] Player square appears cyan
- [ ] Player falls due to gravity
- [ ] Player stops at floor/ceiling

**Phase 3: Input**
- [ ] Keyboard input flips gravity
- [ ] Mouse click flips gravity
- [ ] Audio plays on flip (different pitches)
- [ ] Can flip rapidly without issues

**Phase 4-5: Obstacles & Collision**
- [ ] Obstacles spawn from right
- [ ] Gaps alternate top/bottom
- [ ] Player can pass through gaps
- [ ] Collision triggers game over
- [ ] Game over stops gameplay
- [ ] Restart works correctly

**Phase 6-7: Progression & Polish**
- [ ] Score increases continuously
- [ ] Gaps narrow as score increases
- [ ] High score saves and loads
- [ ] Visual polish elements render
- [ ] No performance issues at 60fps

### Edge Cases to Test

1. **Rapid Input Spam:** Hold down key - should flip repeatedly without breaking
2. **Boundary Collision:** Hit exact top/bottom - should clamp correctly
3. **Zero Gap:** At very high scores - ensure GAP_MIN is passable
4. **First Obstacle:** Easy enough for new players to understand mechanic
5. **localStorage Disabled:** Game should still run without high score
6. **AudioContext Blocked:** Game should work silently if audio fails

### Performance Verification

- Open browser DevTools → Performance tab
- Record 30 seconds of gameplay
- Verify:
  - Consistent 60 FPS
  - Frame time < 16.67ms
  - No memory leaks (obstacles array cleaned up)
  - No jank on obstacle spawn

## Risk Areas & Mitigation

### Risk 1: LOC Constraint Exceeded
**Likelihood:** Medium
**Impact:** High (fails jam requirement)
**Mitigation:**
- Track line count after each phase
- Prioritize features: Core mechanics > Polish
- Use single-letter variable names if desperate (e.g., `o` for obstacles)
- Minify by removing whitespace, combining statements
- Remove audio if necessary (stretch goal)

### Risk 2: AudioContext Autoplay Policy
**Likelihood:** High
**Impact:** Medium (audio missing)
**Mitigation:**
- Initialize AudioContext on first user interaction
- Wrap audio code in try-catch
- Game remains playable without audio
- Test in Chrome, Firefox, Safari

### Risk 3: Collision Detection False Positives
**Likelihood:** Low
**Impact:** High (unfair gameplay)
**Mitigation:**
- Use exact AABB (Axis-Aligned Bounding Box) math
- Verify hitboxes visually (draw rectangles in debug mode)
- Ensure player size constant matches rendering
- Test at boundaries of gaps

### Risk 4: Difficulty Curve Too Hard/Easy
**Likelihood:** Medium
**Impact:** Medium (poor player experience)
**Mitigation:**
- Playtest after Phase 6
- Adjust constants: GAP_BASE, GAP_MIN, score multiplier
- Ensure first obstacle is trivially easy
- Verify 30+ second runs are achievable

### Risk 5: Performance Issues on Older Browsers
**Likelihood:** Low
**Impact:** Low (jam targets modern browsers)
**Mitigation:**
- Avoid excessive canvas operations
- Limit obstacle count (natural via spawn interval)
- Test in Firefox and Safari (not just Chrome)
- Use fillRect (fast) instead of paths

## Implementation Order Summary

1. **Canvas Setup** (25 lines) - Verify rendering works
2. **Player Physics** (30 lines) - Verify falling and rotation
3. **Input & Flip** (15 lines) - Verify control response
4. **Obstacles** (35 lines) - Verify scrolling and gaps
5. **Collision** (25 lines) - Verify game over and restart
6. **Score System** (20 lines) - Verify progression
7. **Polish** (40 lines) - Verify final aesthetic

**Total Estimated:** 190 lines (10 line buffer for adjustments)

## Critical Success Factors

1. **Stay Under 200 LOC:** Cut features before exceeding limit
2. **Responsive Input:** Zero perceived lag on flip
3. **Fair Collision:** Hitboxes match visuals exactly
4. **Clear Progression:** Gap narrowing feels gradual, not sudden
5. **Instant Restart:** No delay after game over input

## Jam Timeline Fit

**Total Implementation Time:** 2-3 hours
- Phase 1-2: 30 minutes (basic setup)
- Phase 3-4: 45 minutes (core mechanics)
- Phase 5-6: 45 minutes (game loop complete)
- Phase 7: 30 minutes (polish)
- Testing: 30 minutes (playtesting and tweaks)

**Fits comfortably within jam constraints.** No external assets means zero download/setup time. Single file means zero build configuration. Can be developed and tested in browser console directly.

## Final Notes for Senior Developer

**Code Style for LOC Efficiency:**
- Use short variable names where unambiguous (e.g., `obs` for obstacles)
- Combine declarations: `let x = 0, y = 0, z = 0;`
- Use ternary operators: `x = a ? b : c;`
- Inline simple calculations: `playerY += (playerVY += GRAVITY);`
- Avoid unnecessary braces for single-line blocks

**Debugging Tips:**
- Add `console.log(frameCount, score)` temporarily for state verification
- Use `ctx.strokeRect()` to visualize hitboxes during testing
- Comment out audio code if it causes issues initially

**Priority Order (if LOC tight):**
1. Core mechanics (gravity flip, obstacles, collision) - MUST HAVE
2. Score system - MUST HAVE
3. Game over/restart - MUST HAVE
4. Audio - NICE TO HAVE
5. Visual polish (glow, rotation) - NICE TO HAVE
6. Intro text - NICE TO HAVE

**The game is shippable after Phase 5.** Phases 6-7 are polish.
