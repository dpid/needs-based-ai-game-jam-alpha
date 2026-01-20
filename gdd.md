# Game Design Document: Gravity Flip

## Elevator Pitch
Navigate an endless vertical corridor by flipping gravity with a single button, dodging obstacles that grow increasingly chaotic. One button. Two directions. Infinite challenge.

## Core Fantasy
You are a small square defying the laws of physics in a minimalist void, flipping between ceiling and floor in a hypnotic rhythm, pushing your reflexes to the limit.

## Theme Integration
**Minimalism - do more with less:**
- Single input creates complex movement patterns
- Simple geometric shapes generate visual depth through motion
- One mechanic (gravity flip) enables varied obstacle patterns
- Minimal colors create maximum contrast and focus
- Complexity emerges from simplicity

---

## Mechanics

### Core Loop
1. Square falls toward bottom of screen
2. Player presses ANY KEY or CLICKS to flip gravity
3. Square now falls toward top of screen
4. Obstacles scroll from right to left
5. Collision = Game Over
6. Score increases over time survived

### Primary Mechanics

1. **Gravity Flip:** Press any key or click to instantly reverse gravity direction. The square immediately changes vertical velocity, creating a sharp angle in trajectory. No cooldown - can be spammed for fine control or used sparingly for swooping arcs.

2. **Endless Scrolling:** The vertical corridor scrolls horizontally at constant speed. Obstacles appear from the right edge with varying configurations. Speed never changes (avoids complexity), but obstacle density increases.

3. **Collision Detection:** Any contact between the square (player) and obstacles (rectangles) triggers game over. Precise hitboxes - what you see is what you get.

### Secondary Mechanics

- **Score Accumulation:** +1 point every 100ms survived. Display updates in real-time at top-left.
- **Obstacle Generation:** Gaps alternate between top and bottom, forcing gravity flips. Gap size decreases over time (score-based).
- **Visual Feedback:** Square rotates based on vertical velocity to show direction of gravity.

### Controls
- **ANY KEY** or **MOUSE CLICK** = Flip Gravity
- Works mid-air with no restrictions

---

## Aesthetics

### Visual Style
**Stark Geometric Minimalism:**
- Black background (#000000)
- White obstacles (#FFFFFF)
- Cyan player square (#00FFFF) with slight glow effect
- Thin white lines marking top/bottom boundaries

### Audio Direction
**Bonus Challenge (Web Audio API):**
- Clean sine wave "blip" on gravity flip (pitch alternates: high for up, low for down)
- Low rumble on collision
- Subtle rising tone as score increases (creates tension)

### Mood/Atmosphere
Hypnotic, meditative focus interrupted by sudden failures. The rhythm of flipping creates a flow state. Clean visuals reduce cognitive load, allowing pure reaction.

---

## Player Experience

### Onboarding
**First 3 Seconds:**
- Square appears mid-screen, falling slowly
- Large text overlay: "PRESS ANY KEY TO FLIP"
- First obstacle appears with generous gap
- Text fades after first input

**Learning Curve:**
- First 10 seconds: Wide gaps, slow introduction
- 10-30 seconds: Gaps narrow, alternating pattern clear
- 30+ seconds: Tight gaps, requires rhythm

### Difficulty Curve
Score-based progression:
- **0-500 points:** Wide gaps (150px), sparse obstacles
- **500-1000 points:** Medium gaps (100px), regular obstacles
- **1000+ points:** Tight gaps (75px), dense obstacles

Never change scroll speed to maintain consistent rhythm.

### Emotional Arc
1. **Curiosity** (0-10s): "Oh, I just flip gravity?"
2. **Confidence** (10-30s): "I've got this pattern down"
3. **Flow State** (30-60s): Muscle memory takes over
4. **Tension** (60s+): Gaps shrink, mistakes feel imminent
5. **Catharsis** (Death): "One more try!"

### Session Length
- Average session: 20-40 seconds
- Skilled players: 60-90 seconds
- Perfect for rapid retry loop

---

## Scope

### MVP Features (Must Have)
- [x] Player square with gravity flip on input
- [x] Vertical velocity with smooth acceleration
- [x] Horizontal obstacle scrolling
- [x] Collision detection and game over
- [x] Real-time score display
- [x] Restart on game over (press key)
- [x] Procedural obstacle generation with gaps
- [x] Score-based difficulty scaling

### Stretch Goals (Nice to Have)
- [x] Player rotation based on velocity direction
- [x] Web Audio API sound effects (bonus challenge)
- [x] High score persistence (localStorage)
- [ ] Visual particle trail on flip
- [ ] Screen shake on collision

### Out of Scope
- Multiple lives
- Power-ups
- Multiple game modes
- Leaderboards (no backend)
- Mobile touch controls (desktop focus for 200 LOC)

---

## Jam Criteria Alignment

| Criterion | How We Address It | Confidence |
|-----------|-------------------|------------|
| Theme Adherence | Single button creates complex emergent gameplay. Minimal visuals, maximum depth. Perfect theme fit. | High |
| Gameplay | Tight, responsive control. Clear fail state. Addictive retry loop. High skill ceiling. | High |
| Polish | Smooth animations, clean visuals, optional audio. Immediate feedback on all actions. | High |
| Creativity | Gravity flip mechanic is proven but allows unique obstacle design. Audio pitch variation adds character. | Medium |
| Technical Merit | Efficient procedural generation. Clean collision logic. All in <200 LOC. Bonus audio implementation. | High |

---

## Technical Considerations

### Tech Stack Alignment
**Perfect fit for vanilla HTML5 Canvas + JavaScript:**
- Canvas 2D context handles all rendering
- Simple physics (gravity, velocity) = basic math
- RequestAnimationFrame for smooth 60fps
- Web Audio API for bonus sounds (optional, clean fallback)
- localStorage for high score (2 lines)

### Performance Targets
- **60 FPS** solid on any modern browser
- **<5ms** per frame render time
- **Instant** input response (<16ms)
- **Zero** load time (no assets)

### Platform Requirements
- Desktop browsers (Chrome, Firefox, Safari, Edge)
- Minimum 800x600 viewport
- Keyboard or mouse required
- No mobile optimization (keeps scope tight)

---

## Implementation Notes for Architect

### Game State Machine
```
READY -> PLAYING -> GAME_OVER -> READY
```

### Key Constants
```javascript
const GRAVITY = 0.5;           // Acceleration per frame
const TERMINAL_VELOCITY = 8;   // Max fall speed
const PLAYER_SIZE = 20;        // Square dimensions
const OBSTACLE_WIDTH = 40;     // Bar width
const OBSTACLE_SPEED = 3;      // Scroll speed
const GAP_BASE = 150;          // Starting gap size
const GAP_MIN = 75;            // Minimum gap at high scores
```

### Obstacle Data Structure
```javascript
{ x: number, gapY: number, gapHeight: number }
```

### Critical Code Sections
1. **Input Handler:** Single event listener for keydown + click
2. **Physics Update:** velocity += gravity, y += velocity, clamp to bounds
3. **Obstacle Generation:** Spawn when last.x < threshold, random/alternating gaps
4. **Collision Logic:** AABB rectangle intersection
5. **Render Loop:** Clear, draw obstacles, draw player (rotated), draw UI

### Audio Implementation (Bonus)
```javascript
const audioCtx = new AudioContext();
function playFlip(goingUp) {
  const osc = audioCtx.createOscillator();
  osc.frequency.value = goingUp ? 440 : 220;
  // ...quick attack/release envelope
}
```

---

## Design Confidence

**High Confidence This Design Will:**
- Be completable in <200 LOC
- Feel responsive and fair
- Showcase theme effectively
- Score well across all criteria
- Support bonus audio challenge

**Core Innovation:**
The gravity flip mechanic combined with score-based gap narrowing creates a difficulty curve that feels natural. The single input allows players to focus entirely on timing, creating a meditative flow state that breaks dramatically on failure - perfect retry psychology.

---

**Ready for Architecture Phase**
