# Testing

## Status of this release

| Evidence | When | Result |
| --- | --- | --- |
| Live Pages probe (`curl`) | 2026-11-03 | **200** |
| Headless Chromium DOM render, `file://` | 2026-11-03 | **canvas present, 960 × 600 backing store** |
| Headless screenshot (1200 × 800) | 2026-11-03 | **rendered, 277 KB PNG** |
| Manual gameplay matrix (§3) | — | **Not run** in this release |
| Touch / Safari / Firefox / mobile | — | **Not verified** |

The documentation commit changed Markdown only; `index.html` is untouched. The headless
pass proves the page boots and paints, not that the game plays well. Anything involving
scoring, AI difficulty or input needs §3.

There is no CI and no committed harness. Screenshot comparison is the only automated
signal, and it cannot judge gameplay.

## 1. Commands

```bash
cd ~/Projects/classic_pong

# live
curl -s -o /dev/null -w '%{http_code}\n' https://somphol.github.io/classic_pong/

# headless DOM: expect one <canvas id="game" width="960" height="600" …>
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new --disable-gpu --virtual-time-budget=3000 \
  --dump-dom "file://$PWD/index.html" | grep '<canvas'

# headless screenshot: expect a non-trivial PNG, not a blank page
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new --disable-gpu --virtual-time-budget=3000 \
  --window-size=1200,800 --screenshot=/tmp/pong.png \
  "file://$PWD/index.html"
ls -l /tmp/pong.png

# local play
python3 -m http.server 8000     # http://localhost:8000/
```

## 2. Static results (2026-11-03)

- Live site: HTTP `200`.
- Headless DOM: `<canvas id="game" width="960" height="600" role="img"
  aria-label="Pong: you control the left paddle, the computer controls the right
  paddle.">` — present with the expected logical size (dpr 1 in headless).
- Screenshot: 1200 × 800, **276,876 bytes** — i.e. the scene actually drew (a blank
  canvas of that size compresses far smaller).
- No console-visible dependency: the file references no external URL, font or image.

## 3. Manual gameplay matrix

Play in a desktop browser, once from `file://` and once over
`http://localhost:8000/`.

| # | Check | Expected |
| --- | --- | --- |
| 1 | Load the page | neon court, net, `YOU`/`CPU` scores at 0, "serve" plate |
| 2 | Click / `Space` | ball serves toward you and play begins |
| 3 | Drag the pointer up and down | left paddle follows; no jump on release |
| 4 | Press `↑`/`↓` and `W`/`S` | paddle moves at key speed; holding works |
| 5 | Let the ball pass you | CPU scores, flash, 0.75 s pause, ball serves to you |
| 6 | Hit with the paddle edge versus centre | visibly different return angles |
| 7 | Rally for a while | ball speeds up; AI speeds up but still misses long rallies |
| 8 | Reach 7 points either side | winner plate; `Space` or click restarts |
| 9 | Press `P` mid-rally | pause plate; `P` resumes from the same state |
| 10 | Press `R` | score resets to 0–0 and returns to serve |
| 11 | Switch tabs mid-rally | game auto-pauses; no giant ball jump on return |
| 12 | Resize the window, then use a HiDPI display | court keeps 1.6 aspect, sharp edges |
| 13 | Play at 120 Hz (or throttle to 30 fps) | speed is the same; no frame-rate dependence |
| 14 | Touch-drag on a phone or emulation | paddle tracks the finger |

## 4. Failure triage

- **Ball tunnels through a paddle** — check the substep count in `moveBall`; it must
  scale with distance.
- **Game runs at double/half speed** — something started using per-frame constants
  instead of `dt`.
- **Paddle sticks after pointer release** — `player.target` was not cleared on
  `pointerup`/`pointercancel`.
- **AI unbeatable** — `MAX_SPEED`, the `rally × 22` term or the easing factor changed.
- **Blank page** — canvas sizing or a script error before the first `requestAnimationFrame`.

## 5. Not verified

- Real touch devices, Safari, Firefox, Edge.
- Accessibility with a screen reader (the canvas has an `aria-label` and nothing more;
  there is no alternative text interface).
- Long sessions: no soak test, and no memory check on the particle/trail arrays.
