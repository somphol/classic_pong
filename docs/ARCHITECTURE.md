# Architecture

A single canvas game. No modules, no build, no dependencies. This map is enough to make
a change without reading all 730 lines.

## 1. Document layout

| Lines | Block | Contents |
| --- | --- | --- |
| 1–89 | CSS | reset, page backdrop, CRT scanline + vignette overlay, canvas sizing |
| 109–116 | Markup | `<canvas id="game">` with `role="img"` and a descriptive `aria-label` |
| 117–728 | Script | one IIFE; constants, state, input, update, physics, FX, drawing, loop |

The CSS does no gameplay work. The canvas is sized by `width: min(94vw, 1100px,
calc(86vh * 1.6))` with a fixed 1.6 aspect ratio, so the layout is resolution-independent.

## 2. Coordinate system and resolution

- Logical space is **960 × 600**, origin top-left, `y` down.
- `resize()` sets `canvas.width/height` to `W/H × dpr` (dpr capped at 2) and
  `ctx.setTransform(dpr, …)`. All game maths stays in logical units.
- CSS scales the element; the browser never sees logical coordinates.

## 3. State

```js
player = { x, y, w, h, flash, target }   // target = pointer-controlled centre Y
ai     = { x, y, w, h, flash }
ball   = { x, y, vx, vy, r, speed, trail }
state  = { mode, score:[you,cpu], timer, lastScorer, winner, flash, scorePulse }
```

State machine:

| Mode | Meaning | Exit |
| --- | --- | --- |
| `serve` | ball rests on the serving paddle | `action()` → `play` |
| `play` | live rally | point scored → `point`; 7 reached → `over` |
| `point` | 0.75 s pause with scoring flash | `serve` for the conceded side |
| `over` | match finished | `action()`/`resetMatch()` → `serve` |

Additional module-level variables: `particles`, `rally`, `aiTargetY`, `paused`,
`control` (last input device), `keys` (held keys), `last` (timestamp).

## 4. Update pipeline

`frame(now)`:

1. `dt = (now - last) / 1000`, clamped to `0.05` s.
2. If not paused, `step(dt)`.
3. `draw()`.
4. `requestAnimationFrame(frame)`.

`step(dt)` calls `updatePlayer`, `updateAI`, `updateFx`, then either `moveBall` (adding a
trail point) or the point timer that leads back to `serve`.

## 5. Physics

- **Paddle hit** — `hitPaddle(p, dir, color)` checks circle/rect overlap, pushes the ball
  clear, then sets the outgoing angle from where it struck the paddle, bounded by
  `MAX_BOUNCE_ANGLE` (60°) and scaled by `speed`.
- **Speed ramp** — each paddle hit multiplies `ball.speed` by `SPEED_GAIN` (1.045), from
  `BASE_SPEED` (430) to `MAX_SPEED` (1180) px/s.
- **Substepping** — `moveBall` splits the delta into `ceil(distance / (r × 0.9))` steps;
  wall bounces, paddle hits and point detection all happen per substep. This is why a
  ball at 1180 px/s cannot pass through a 14 px paddle.
- **Walls** — top and bottom bounce with a small particle burst.
- **Points** — past `x < -3r` or `x > W + 3r`, `point(scorer)` updates the score, flashes,
  and sets `mode = 'point'` with a 0.75 s timer.

## 6. AI

`predictY()` projects the ball to the AI's plane and reflects the result across the
walls (`m` modulo `2 × span`). `updateAI()`:

- targets the centre when the ball is not approaching;
- adds `sin(now / 420) × 10` px of wobble;
- eases `aiTargetY` toward the target with `1 - exp(-dt × 9)` (reaction lag);
- moves at most `430 + min(250, rally × 22)` px/s, clamped to the court.

Lower the speed cap or easing factor to make it stronger; raise them to make it weaker.

## 7. Rendering order

`draw()` → `drawBackground` (radial vignette), `drawNet` (dashed centre line), `drawScores`
(`YOU` / `CPU`, pulsing on score), `drawPaddles` (with flash glow), `drawBall` or
`drawServeBall`, `drawParticles`, then `drawOverlay` (`plate()` renders the serve prompt,
pause plate or winner plate).

The CRT effect — scanlines, vignette, subtle flicker — is CSS `body::before` plus a
page-level gradient, not canvas post-processing. Removing it costs nothing in gameplay
accuracy but changes the art direction.

## 8. Input

| Input | Handler | Effect |
| --- | --- | --- |
| Pointer/touch move | `pointermove` → `pointerToY` | sets `player.target` |
| Pointer down / tap | `pointerdown` | `action()` — serve or restart |
| `↑` / `W`, `↓` / `S` | `keydown` (held in `keys`) | moves the paddle at key speed |
| `Space` | `action()` | serve, or restart when `over` |
| `P` / `Esc` | `togglePause()` | pause/resume during `play`/`point` |
| `R` | `resetMatch()` | zero the score and return to `serve` |

`control` tracks whether the last move came from pointer or keys, so the paddle does not
snap back to a stale pointer target.

## 9. Change guide

| To change… | Touch |
| --- | --- |
| Court size or aspect | `W`, `H`, the canvas CSS aspect ratio, and the overlay coordinates |
| Difficulty | `MAX_SPEED`, `SPEED_GAIN`, `updateAI` easing and speed cap |
| Winning score | `WIN_SCORE` (and the README) |
| Palette | `COLORS` and the CSS custom properties |
| Serve delay | the `0.75` timer in `point()` |
| Pause rules | `togglePause()` and the `document.hidden` listener |
