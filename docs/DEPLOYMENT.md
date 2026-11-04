# Deployment

Hosted on **GitHub Pages**, served directly from the repository. No build output, no
workflow: the committed `index.html` on `main` is what players load.

- Live URL: <https://somphol.github.io/classic_pong/>
- Repository: <https://github.com/somphol/classic_pong>
- Source branch: `main`, folder `/` (root)
- Entry file: `index.html`

## 1. Deploy a change

```bash
cd ~/Projects/classic_pong

# verify locally first
python3 -m http.server 8000     # http://localhost:8000/
# run the matrix in docs/TESTING.md

git add -A
git commit -m "…"
git push origin main
```

Confirm the result:

```bash
curl -s -o /dev/null -w '%{http_code}\n' https://somphol.github.io/classic_pong/
```

Pages usually updates within a minute. If the old version persists, hard-refresh; there
is no service worker to unregister.

## 2. Repository settings that matter

| Setting | Value | Why |
| --- | --- | --- |
| Pages source | branch `main`, `/ (root)` | the game is a single root file |
| Custom domain | none | default `*.github.io` host |
| Enforce HTTPS | on | default certificate |
| Actions | none | `.github/workflows/` does not exist |

`.nojekyll` is committed and must stay: it prevents Jekyll processing on the published
tree.

## 3. Published files

The whole root is public: `index.html`, `screenshot.png`, `docs/social-card.png`,
`README.md`, `CHANGELOG.md`, `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`,
`LICENSE`, `AGENTS.md`, `docs/*.md`, `robots.txt`, `sitemap.xml`.

No secrets, no environment variables, no runtime configuration. The game makes no network
requests at all — if you see one in devtools after a change, that change broke the
offline guarantee.

## 4. Rollback

```bash
git revert <bad-commit>        # preferred
git push origin main
```

Avoid `git reset --hard` + force-push on `main` unless the bad commit must disappear from
history; Pages serves whatever `main` points at.

## 5. Caching and assets

Pages serves static files with ETags and short caching. The only binary assets are the two
PNGs used by the README and social preview; the game itself loads nothing. If a deploy
looks stale, check the commit SHA in the page source or use a private window.

## 6. Local versus production

`file://` works and is the quickest smoke test. Use the HTTP server when checking
pointer capture, keyboard focus or anything that depends on document origin; those are
the conditions the release was checked under.
