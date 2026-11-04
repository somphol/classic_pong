# Reproduction book

How to rebuild **Classic Pong** from scratch, or re-verify that a rebuild matches the
release. Read [ARCHITECTURE.md](ARCHITECTURE.md) first for the map.

> **The source is the truth.** `index.html` on `main` is the release. Rebuild only when
> the file is lost or you are deliberately making a variant.

## 1. What is being reproduced

A single-file neon **Pong**:

- **Court** — fixed 960 × 600 logical space, scaled by CSS to fit the viewport at a 1.6
  aspect ratio; DPR-backed canvas (capped at 2).
- **Play** — left paddle is the player, right is a predictive AI. First to **7**.
- **Physics** — delta-time movement, speed ramps 1.045× per paddle hit from 430 to
  1180 px/s, angle set by contact point (≤ 60°), per-substep collision.
- **AI** — wall-reflecting path prediction, reaction easing, speed cap that grows with
  rally length, small sine wobble.
- **Visuals** — CRT scanlines and vignette in CSS; canvas-drawn net, glowing paddles,
  ball trail, hit particles, score pulse, overlay plates.
- **Input** — pointer/touch drag, `↑`/`↓`/`W`/`S`, `Space`, `P`/`Esc`, `R`.
- **Delivery** — one HTML file, no dependencies, no assets, no network calls, no audio.
  GitHub Pages, `main` / root.
- **Release** — v1.0.0.

## 2. Requirements matrix

| # | Requirement | Implementation | Where |
| --- | --- | --- | --- |
| 1 | Pong on canvas, neon retro look | `draw*` functions, CSS scanlines | script + CSS |
| 2 | Player paddle via mouse/touch/keyboard | `pointerToY`, key handlers, `updatePlayer` | script |
| 3 | AI opponent that can be beaten | `predictY`, `updateAI` speed cap | script |
| 4 | Ball bounces and accelerates | `hitPaddle`, `SPEED_GAIN`, `MAX_SPEED` | script |
| 5 | Contact point changes return angle | angle branch in `hitPaddle` | script |
| 6 | No tunnelling at high speed | substeps in `moveBall` | script |
| 7 | Score, first to 7, restart | `point`, `WIN_SCORE`, `resetMatch` | script |
| 8 | Serve / pause / reset controls | `action`, `togglePause`, `resetMatch` | script |
| 9 | Fixed logical resolution, crisp on HiDPI | `resize` with dpr cap | script + CSS |
| 10 | Static hosting | `.nojekyll`, `robots.txt`, `sitemap.xml` | repo root |

## 3. Build order

1. **Shell** — head, CSS backdrop, canvas element with fixed 960 × 600 attributes and an
   `aria-label`.
2. **Boot** — IIFE, canvas/context, `resize()` with DPR handling, the `frame` loop with
   `dt` clamping.
3. **State** — player, AI, ball, and the `serve | play | point | over` machine.
4. **Movement** — `updatePlayer` (pointer target and key speed), then `updateAI` using a
   naive straight-line target; refine to wall reflection afterwards.
5. **Physics** — `moveBall` substeps, wall bounces, `hitPaddle` angle and speed ramp,
   scoring at the edges.
6. **Scoring flow** — `point`, the 0.75 s pause, serve direction.
7. **Presentation** — background, net, scores, paddles, ball, trail, particles, overlay
   plates, then the CSS CRT overlay.
8. **Feel** — flash values, score pulse, AI wobble.
9. **Metadata and docs** — README, CHANGELOG, social card, sitemap, this book.

## 4. Acceptance checks

A rebuild is done when, in desktop Chromium:

1. The court renders at 1.6 aspect and fills the window without cropping.
2. Serve starts play; the ball is visible and leaves a trail.
3. Pointer drag moves the paddle smoothly and does not snap on release.
4. Keys move the paddle; holding a key does not stutter.
5. Edge hits return steeper than centre hits.
6. Ball speed rises over a rally and never exceeds a playable speed.
7. AI returns most balls early, misses more as the rally grows.
8. A point pauses play briefly, flashes, then serves to the conceding side.
9. At 7 points the winner plate appears and `Space` restarts a fresh match.
10. `P`/`Esc` pauses and resumes; `R` resets; hiding the tab auto-pauses.
11. A 30 fps and a 120 Hz display produce the same ball speed.

Record the run in [TESTING.md](TESTING.md).

## 5. Provenance

| Item | Value |
| --- | --- |
| Initial build | `384b5cd` "Add classic Pong arcade game" |
| Live link | `aa07809` |
| Social card + README header | `43584eb` |
| Repo docs and community files | `0a85abc` |
| Release docs | this commit (`AGENTS.md` + `docs/`) |
| License | MIT |

`CHANGELOG.md` is the version record; do not invent history beyond it.
