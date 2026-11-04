# AGENTS.md — CLASSIC PONG

> Handoff for the next maintainer or agent. Rebuild detail is in
> [docs/REPRODUCTION.md](docs/REPRODUCTION.md); the technical map is
> [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## 1. What this is

A neon-retro **Pong** game in one HTML file: HTML5 Canvas plus vanilla JavaScript, no
libraries, no assets, no network calls, no audio. The player defends the left paddle
against a predictive AI on the right; first to **7** points wins. Matching is done at a
fixed 960 × 600 logical resolution, scaled by CSS, with the backing store multiplied by
device pixel ratio (capped at 2).

It is **not** a multiplayer or networked game, not a mobile app, and not a framework
starter. Live copy: <https://somphol.github.io/classic_pong/>.
Local master: `~/Projects/classic_pong`. Current release: **v1.0.0**.

## 2. Commands

```bash
cd ~/Projects/classic_pong

# run (no install, no build)
open index.html
python3 -m http.server 8000        # http://localhost:8000/

# verify (macOS, Chrome installed)
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new --disable-gpu --virtual-time-budget=3000 \
  --dump-dom "file://$PWD/index.html" | grep '<canvas'
curl -s -o /dev/null -w '%{http_code}\n' https://somphol.github.io/classic_pong/

# deploy — Pages serves main / root; no workflow
git push origin main
```

Tests are manual; the matrix is in [docs/TESTING.md](docs/TESTING.md). There is no CI.

## 3. Architecture

One file, `index.html` (730 lines):

| Lines | Block |
| --- | --- |
| 1–89 | head and CSS: reset, body backdrop, scanline/vignette overlay, canvas sizing |
| 109–116 | body markup: a single `<canvas id="game">` with an `aria-label` |
| 117–728 | one IIFE: constants → state → input → update → physics → FX → drawing → loop |

Key structures:

- **Constants** — `W = 960`, `H = 600`, `PADDLE_W/H`, `BALL_R`, `BASE_SPEED = 430`,
  `MAX_SPEED = 1180`, `SPEED_GAIN = 1.045`, `MAX_BOUNCE_ANGLE = 60°`, `WIN_SCORE = 7`.
- **Objects** — `player`, `ai` and `ball` hold position and velocity; `state` is the
  machine: `mode` ∈ `serve | play | point | over`, plus `score`, `timer`, `winner` and
  flash timers.
- **Loop** — `frame(now)` computes `dt`, clamps it to 50 ms (tab switches, stalls), calls
  `step(dt)` unless paused, then `draw()`, then re-arms `requestAnimationFrame`.
- **Physics** — `moveBall` substeps by `ceil(distance / (radius * 0.9))` so a fast ball
  cannot tunnel through a paddle; collisions are resolved inside the substep loop.
- **AI** — `predictY` reflects the ball's straight-line path across the walls, then
  `updateAI` eases toward that target (`1 - exp(-dt * 9)`) with a speed cap of
  `430 + min(250, rally * 22)` px/s, so the AI loses long rallies by design.
- **Rendering** — background, net, scores, paddles, ball/trail, particles, then the
  overlay plate (serve prompt, pause, winner). All shapes are hand-drawn on canvas.
- **Input** — pointer/touch (drag anywhere to move), `↑`/`↓` and `W`/`S`, `Space` to
  serve or restart, `P`/`Esc` to pause, `R` to reset the match. Auto-pauses when the tab
  is hidden mid-rally.

## 4. Decisions and constraints

- **Single file, zero dependencies.** No fonts, images or audio files; the CRT look is
  CSS gradients plus canvas drawing. Keep it that way.
- **Fixed logical resolution.** Gameplay maths never reads CSS pixels, so behaviour is
  identical at every window size. `resize()` only changes the backing store and
  transform (DPR capped at 2 to bound fill cost).
- **Delta-time, not frame-count.** Speeds are px/s. Do not reintroduce per-frame
  constants; they make the game unplayable at 120 Hz.
- **`dt` clamp of 0.05 s** is deliberate: it prevents a huge jump after a stall from
  teleporting the ball past a paddle.
- **The AI is intentionally imperfect.** Speed cap, reaction easing and a small sine
  wobble are the difficulty, not a placeholder. Raising them makes the game unwinnable.
- **`WIN_SCORE = 7`** is referenced in the README; changing it means updating the docs.

## 5. Known issues

- No automated tests, no CI, no headless gameplay harness. `docs/TESTING.md` records what
  was actually run.
- No audio by design; "arcade feel" is visual only.
- Verified in desktop Chromium only. Touch input, Safari, Firefox and small screens are
  **not verified**.
- The AI's prediction reflects off walls but not off the player paddle, so it can be
  beaten by sharp-angled returns; that is accepted behaviour, not a bug.
- `P`/`Esc` pause only during `play` or `point`; pressing them on the serve screen does
  nothing.

## 6. Continuity

- Live: <https://somphol.github.io/classic_pong/> (HTTP 200 at the time of writing).
- Pages: `main` branch, repository root, `.nojekyll` present, no Actions workflow.
- Local master copy: `~/Projects/classic_pong`.
- First build: `384b5cd` "Add classic Pong arcade game"; see `CHANGELOG.md`.
