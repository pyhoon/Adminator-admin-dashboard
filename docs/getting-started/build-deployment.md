---
layout: default
title: Production build
nav_order: 4
parent: Getting started
---

# Production build & deployment
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Build commands

```bash
# Default — production build, NOT minified (good for debugging output)
npm run build

# Production build with full minification (Terser + cssnano)
npm run release:minified

# Same as `build` but explicit NODE_ENV=production
npm run release:unminified

# Wipe dist/
npm run clean
```

All three production commands first run `npm run clean`, then build into `dist/`.

## What gets emitted

Running `npm run release:minified` produces:

```text
dist/
├── *.html               # 18 minified HTML pages (1.6 KB – 21 KB each)
├── runtime.js           # Webpack runtime, ~1 KB
├── 2026.js              # Our app code, ~24 KB
├── vendor-chartjs.js    # Chart.js, ~190 KB
├── vendor-fullcalendar.js  # FullCalendar, ~250 KB
├── vendors.js           # jsvectormap + other shared deps, ~160 KB
├── style.css            # Extracted CSS, ~90 KB
└── assets/
    └── static/          # Fonts, images, icons (copied as-is from src/)
```

Total ≈ **6.2 MB** uncompressed (the bulk is the static font files; gzipped JS+CSS is around 200 KB).

## Code splitting

Webpack auto-splits the bundle by source. The legacy `main` entry from v3 is gone — there's only one entry now (`2026`). Three vendor chunks are split out for browser caching:

| Chunk | Why split |
|-------|-----------|
| `vendor-chartjs.js` | Heavy enough to deserve its own cache lifetime |
| `vendor-fullcalendar.js` | Same |
| `vendors.js` | Everything else from `node_modules` |

If you don't use Chart.js or FullCalendar on a particular page, the browser still downloads the chunks (HtmlWebpackPlugin injects them globally). For per-page optimization, see "Removing unused integrations" below.

## Bundle analyzer

```bash
npm run build:analyze
```

Opens an interactive treemap of the bundle in your browser — useful for spotting unexpected size growth. The report is also written to `dist/bundle-report.html`.

## Deploying

The `dist/` folder is a static site. Drop it on any static host:

### Netlify

```bash
# Push your repo, then in Netlify dashboard:
#   Build command:    npm run release:minified
#   Publish directory: dist
```

Or via the CLI:

```bash
npm install -g netlify-cli
npm run release:minified
netlify deploy --dir=dist --prod
```

### Vercel

```bash
npm install -g vercel
npm run release:minified
vercel deploy dist --prod
```

Or set up `vercel.json`:

```json
{
  "buildCommand": "npm run release:minified",
  "outputDirectory": "dist",
  "framework": null
}
```

### GitHub Pages

```bash
npm run release:minified
# Push dist/ to a gh-pages branch:
git subtree push --prefix dist origin gh-pages
```

Or use a workflow — see [`.github/workflows/`](https://github.com/puikinsh/Adminator-admin-dashboard/tree/master/.github/workflows) if there's a deploy.yml.

### Cloudflare Pages

```bash
# In Cloudflare Pages → Create project → connect your GitHub repo:
#   Build command:    npm run release:minified
#   Build output:     dist
```

### S3 / CloudFront

```bash
npm run release:minified
aws s3 sync dist/ s3://your-bucket --delete \
  --cache-control "public, max-age=31536000, immutable" \
  --exclude "*.html"
aws s3 sync dist/ s3://your-bucket \
  --cache-control "public, max-age=0, must-revalidate" \
  --exclude "*" --include "*.html"
```

The two-pass sync sets immutable caching for hashed assets (JS/CSS/images) and no-cache for HTML, which is the right CDN strategy.

### Any static host

Webpack output uses relative URLs (no `<base href>` requirement). You can serve `dist/` from any subpath. If your deploy needs absolute paths (e.g. <https://example.com/admin/>), set `output.publicPath` in `webpack/config.js` to `/admin/`.

## Cache strategy

The recommended setup:

- **HTML files** — short TTL (e.g. 5 minutes) or `no-cache`. They reference hashed JS/CSS bundles, so a fresh HTML pull picks up new bundles.
- **JS / CSS bundles** — `immutable` with 1-year TTL. Webpack can produce content-hashed filenames (e.g. `2026.abc123.js`) when configured. To enable, edit `webpack/config.js` and set `output.filename: '[name].[contenthash].js'`.
- **Static assets** (`assets/static/`) — `immutable` with 1-year TTL. Filenames are stable.

The default config doesn't add content hashes (kept simple). Add them if you serve behind a CDN.

## Removing unused integrations

If your dashboard doesn't use FullCalendar or jsvectormap, you can save ~430 KB by removing them:

1. Delete the import from `src/assets/scripts/2026/index.js`:

   ```js
   // Remove these:
   import { initCalendarPage } from './calendar.js';
   import { initVectorMaps } from './maps.js';
   ```

2. Stop calling them in `start()`:

   ```js
   function start() {
     mountShell();
     initShellBehaviors();
     initCharts();
     // initVectorMaps();   // ← removed
     // initCalendarPage(); // ← removed
   }
   ```

3. Delete the corresponding HTML pages (`calendar.html`, `vector-maps.html`) and remove from `webpack/plugins/htmlPlugin.js`.

4. Optionally `npm uninstall @fullcalendar/* jsvectormap` to drop them from `node_modules`.

The next build will exclude both libraries entirely. The remaining `2026.js` + `vendor-chartjs.js` + `vendors.js` totals around **220 KB** minified.

## Production sanity checklist

Before tagging a release:

- [ ] `npm run lint` is 0/0
- [ ] `npm run release:minified` succeeds with 0 warnings
- [ ] `dist/` opens correctly when served by a static server (try `npx serve dist`)
- [ ] Theme toggle works on every page
- [ ] Charts, calendar, and vector map render correctly in both themes
- [ ] All 18 page links from the sidebar resolve (no 404s)

For a quick local preview of the built output:

```bash
npx serve dist -p 5000
```
