---
layout: default
title: Development
nav_order: 3
parent: Getting started
---

# Development workflow
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Dev server

```bash
npm start
```

Webpack dev server on <http://localhost:4000>. Edits to `src/**/*.html`, `src/assets/scripts/2026/*.js`, or `src/assets/styles/2026/*.scss` trigger an auto-rebuild and full-page reload (no HMR for HTML — that's a webpack-dev-server limitation, not a project choice).

For more verbose dev output:

```bash
npm run dev   # webpack-dashboard wrapper
```

## All npm scripts

| Script | What it does |
|--------|--------------|
| `npm start` | Dev server on `:4000` with file-watch + auto-reload |
| `npm run dev` | Dev server with the webpack-dashboard UI |
| `npm run build` | Production build to `dist/`, **un**minified (good for debugging output) |
| `npm run release:minified` | Production build with full minification (Terser + cssnano) |
| `npm run release:unminified` | Same as `build`, explicit `NODE_ENV=production` |
| `npm run preview` | Re-runs the dev server |
| `npm run clean` | Wipes `dist/` |
| `npm run lint` | ESLint + Stylelint, must be 0/0 |
| `npm run lint:js` | ESLint only |
| `npm run lint:scss` | Stylelint only |
| `npm run build:analyze` | Build with `webpack-bundle-analyzer` (opens an interactive treemap) |
| `npm run screenshots` | Generate the README screenshots (requires `npm start` running) |

## Making changes

### Edit a color or any visual token

Open `src/assets/styles/2026/_tokens.scss`. All visual decisions live here — colors, font sizes, spacing, shadows. Light + dark variants are defined under `:root[data-theme="light"]` and `:root[data-theme="dark"]`.

Example — change the brand color:

```scss
:root[data-theme="light"] {
  --primary:       #2563EB;     // ← old
  --primary:       #DB2777;     // ← new (pink)
  --primary-soft:  #FDF2F8;     // bump the soft variant too
}
```

Save the file. The dev server rebuilds (~1 sec) and the page reloads with the new color applied to every component on every page.

### Edit a single page

Pages are static HTML in `src/*.html`. Each page imports nothing — the shell, JS, and CSS are wired in by webpack via HtmlWebpackPlugin. Just edit the page's content (everything inside `<main class="content">`) and save.

### Edit shell behavior (sidebar item, breadcrumbs, etc.)

For sidebar items, edit `src/assets/scripts/2026/Shell.js` (the `NAV` array at the top). For breadcrumbs / which sidebar item is highlighted, set `data-active` and `data-crumbs` on each page's `<body>`.

For runtime behaviors (theme toggle, dropdown click, accordion expand), edit `src/assets/scripts/2026/init.js`.

## Linting

Both linters must be 0 errors / 0 warnings before merging:

```bash
npm run lint
```

ESLint 9 (flat config in `eslint.config.mjs`) handles JS. Stylelint 17 (`.stylelintrc.json`) handles SCSS.

The CI hook on `prepublishOnly` runs both — `npm publish` will refuse to ship a dirty tree.

### Fixing lint failures

Most stylelint issues are auto-fixable:

```bash
npx stylelint "src/**/*.scss" --fix
```

ESLint similarly:

```bash
npx eslint src/ --fix
```

## Tests

Currently none. v3's tests covered the `utils/` modules that were deleted in v4. A new test suite for `init.js`, `Shell.js`, `charts.js`, `calendar.js`, `maps.js` is on the roadmap but not yet shipped.

If you want to write tests, vitest is already in `devDependencies`; create `tests/<thing>.test.js` and run `npm run test` (or `npm run test:run` for one-shot).

## Browser support

Modern evergreen browsers — Chrome, Firefox, Safari, Edge (last 2 versions). The template uses `color-mix()`, `aspect-ratio`, CSS custom properties, and modern flex/grid. IE11 isn't supported and won't be.

If you need to drop a browser version that supports the above, edit the `browserslist` field in `package.json` — Babel and Autoprefixer use it.

## Theme switching while developing

The dev page has a moon/sun icon in the topbar. Click to toggle. The choice persists in `localStorage` (`dash26-theme` key) and `data-theme` flips on `<html>`.

To force a specific starting theme during dev (without clicking), open DevTools console:

```js
localStorage.setItem('dash26-theme', 'dark'); location.reload();
```

## Hot tips

- **Open multiple pages at once.** All 18 pages share the same bundle, so once the dev server is warm, navigation is instant. Have `email.html`, `calendar.html`, `forms.html` open in separate tabs.
- **Inspect a token.** In DevTools → Elements → `<html>`, the Computed pane shows every `--token` value for the active theme. Useful for picking the right one when styling something new.
- **Find which CSS partial owns a class.** Each partial in `src/assets/styles/2026/` is named after its scope (`_chat.scss` for `.chat-*` classes, `_data.scss` for `.data-table` etc.). Grep is fastest: `grep -rn '.btn--primary' src/assets/styles/2026/`.
- **Bundle too big?** Run `npm run build:analyze` and the treemap will show what's bloating it. Most of the bundle is FullCalendar (~250 KB) and Chart.js (~190 KB) — those are real features, not dead weight.
