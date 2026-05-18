# Talimara demo — GitHub Pages deploy notes

This directory is GitHub Pages-ready. Everything inside `dist/` is what you
publish; nothing else is required.

## File inventory

| File | Purpose |
|------|---------|
| `index.html` | The demo + landing. Self-contained, ~33 KB. |
| `og-card.png` | 1200×630 social card (Open Graph + Twitter). |
| `favicon.ico` | Multi-resolution icon for desktop browser tabs. |
| `favicon-16.png` / `favicon-32.png` | PNG fallbacks for tabs. |
| `apple-touch-icon.png` | 180×180 home-screen icon (iOS / Safari). |
| `icon-192.png` / `icon-512.png` | PWA manifest icons. |
| `site.webmanifest` | PWA manifest. |
| `.nojekyll` | Empty marker — disables GitHub Pages' Jekyll preprocessing. |
| `DEPLOY.md` | This file. (Safe to publish; not load-bearing.) |

## Before you publish — swap the link tokens

`index.html` has three placeholder URLs in the top-strip nav. Replace them
with real URLs before deploying:

| Token | Replace with |
|-------|--------------|
| `__WRITEUP_URL__` | The public Kaggle writeup URL (recommended — keep one canonical public writeup; don't render a second copy into `dist/`) |
| `__VIDEO_URL__` | The 3-minute YouTube video link |
| `__REPO_URL__` | The public GitHub repo URL (e.g. `https://github.com/<user>/talimara`) |

Recipe (run inside `dist/`):

```bash
sed -i \
  -e 's|__WRITEUP_URL__|https://www.kaggle.com/competitions/...your-writeup-url...|g' \
  -e 's|__VIDEO_URL__|https://youtu.be/...your-video-id...|g' \
  -e 's|__REPO_URL__|https://github.com/<your-user>/talimara|g' \
  index.html
```

Verify nothing was missed:

```bash
grep -n '__\(WRITEUP\|VIDEO\|REPO\)_URL__' index.html
# (should print nothing)
```

## Publish options (pick one)

### A · Project subdir of your repo (simplest)

1. Create a `gh-pages` branch *or* configure Pages to serve from the
   `/docs` folder of `main`.
2. Copy the contents of this `dist/` directory into the chosen location.
3. In repo Settings → Pages, set the source.

URL will be: `https://<user>.github.io/<repo>/`

### B · Dedicated `<user>.github.io` repo (cleaner URL, separate repo)

1. Create `<user>.github.io` repo.
2. Put the contents of `dist/` at the root.
3. Pages turns on automatically.

URL will be: `https://<user>.github.io/`

### C · Custom domain

Add a `CNAME` file with your domain, point DNS at `<user>.github.io`, and
flip the HTTPS toggle in Pages settings once GitHub provisions a cert.

## What the demo does at runtime

- Loads React + Babel from unpkg, MediaPipe LLM Inference Web from
  JSDelivr. (Preconnect hints are in `index.html` `<head>`.)
- If WebGPU is available and the user clicks **Load Gemma 4 (2 GB · WebGPU)**:
  fetches `gemma-4-E2B-it-litert-lm` from HuggingFace and grades in-browser.
- Otherwise: serves canned grades captured from a real Android-device run
  on a Samsung Galaxy A15 5G. A yellow banner makes this honest.

GitHub Pages serves HTTPS by default on `*.github.io`, so WebGPU's
secure-context requirement is met. HuggingFace serves
`Access-Control-Allow-Origin: *` on `resolve/main/<file>`, so the 2 GB
model fetch works cross-origin.

## What NOT to commit

The `gemma-4-E2B-it-litert-lm.task` file is ~2 GB. Do not commit it.
GitHub's individual-file limit is 100 MB and the soft repo cap is 1 GB.
The demo fetches it from HuggingFace at runtime; that is intentional.

## Verifying locally before deploy

```bash
cd dist
python3 -m http.server 8000
# open http://localhost:8000
```

Check the share-card preview in:
- https://www.opengraph.xyz/  (paste your URL after deploy)
- https://cards-dev.twitter.com/validator
