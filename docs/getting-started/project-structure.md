---
layout: default
title: Project structure
nav_order: 2
parent: Getting started
---

# Project structure
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Top level

```text
adminator/
├── src/                     # All source code (HTML, JS, SCSS, static assets)
├── docs/                    # This documentation site (Jekyll, GitHub Pages)
├── webpack/                 # Webpack config split into rules + plugins
├── screenshots/             # Documentation + listing screenshots
├── scripts/                 # Build / publish helper scripts
├── dist/                    # Production build output (gitignored, created by `npm run build`)
├── package.json
├── webpack.config.js        # Thin re-export of webpack/config.js
├── eslint.config.mjs        # ESLint 9 flat config
├── .stylelintrc.json
├── README.md
├── CHANGELOG.md
├── LICENSE
└── CLAUDE.md                # Notes for AI assistants working in this repo
```

## `src/` — everything you'll actually edit

```text
src/
├── *.html                   # 18 pages, each ~500 lines (mostly content)
└── assets/
    ├── scripts/2026/        # The only JS — 6 modules, ~36 KB total
    │   ├── index.js         # Entry: imports SCSS, mounts shell, runs init
    │   ├── Shell.js         # NAV manifest + sidebar/topbar/footer renderers
    │   ├── init.js          # Theme toggle, dropdowns, nav-groups, todos, accordions, tabs
    │   ├── charts.js        # Chart.js seeds, theme-aware
    │   ├── calendar.js      # FullCalendar seed events + toolbar binding
    │   └── maps.js          # jsvectormap world map
    ├── styles/2026/         # The only SCSS — 18 partials, ~280 KB before minification
    │   ├── index.scss       # Entry: @use's everything below
    │   ├── _tokens.scss     # CSS variables (light + dark)
    │   ├── _base.scss       # Reset, body, .eyebrow, .mono, font import
    │   ├── _animations.scss # rise-in / bar-in / fade-in / draw / spin
    │   ├── _shell.scss      # .shell, sidebar, topbar, footer chrome
    │   ├── _dropdowns.scss  # .dd-* (notifications, messages, profile)
    │   ├── _components.scss # .hero, .btn, .card, .grid, .table, .tag
    │   ├── _forms.scss      # inputs, select, textarea, check, radio, switch
    │   ├── _ui.scss         # alerts, badges, progress, spinner, tabs, accordion, modal
    │   ├── _auth.scss       # signin/signup split-screen shell
    │   ├── _error.scss      # 404 / 500 cards
    │   ├── _chat.scss       # 2-pane chat layout
    │   ├── _data.scss       # data-table, pager
    │   ├── _charts.scss     # chart-canvas-wrap, legend, meta row
    │   ├── _dashboard.scss  # KPIs, sv-* (site visits), todo, weather
    │   ├── _email.scss      # 3-pane email layout
    │   ├── _calendar.scss   # Mini-cal, rail, upcoming list
    │   ├── _fullcalendar.scss  # FullCalendar token overrides
    │   └── _responsive.scss # All media queries in one place
    └── static/              # Fonts, images, icons (copied to dist/ as-is)
```

There are **no other JS or SCSS folders**. The legacy `scripts/components/`, `scripts/utils/`, `styles/spec/`, and `styles/vendor/` from v3 were all deleted in v4.

## `webpack/` — build configuration

```text
webpack/
├── config.js                # Main entry — reads manifest, assembles entry/optimization
├── manifest.js              # Path constants + env detection
├── devServer.js             # Dev server config (port, hot reload, fallback)
├── rules/
│   ├── index.js
│   ├── js.js                # Babel
│   ├── sass.js              # sass + postcss + css-loader / mini-css-extract
│   ├── css.js               # Plain CSS
│   ├── images.js            # Asset modules
│   └── fonts.js             # Asset modules
└── plugins/
    ├── index.js             # Plugin assembly
    ├── htmlPlugin.js        # HtmlWebpackPlugin per page (one per HTML file in src/)
    ├── extractPlugin.js     # MiniCssExtractPlugin (production only)
    ├── copyPlugin.js        # Copies src/assets/static/ → dist/assets/static/
    ├── caseSensitivePlugin.js
    ├── dashboardPlugin.js   # webpack-dashboard (dev only, npm run dev)
    └── internal.js          # HMR plugin in dev
```

Single webpack entry: `2026: src/assets/scripts/2026/index.js`. Produces `runtime.js`, `2026.js`, plus auto-split `vendor-fullcalendar.js`, `vendor-chartjs.js`, `vendors.js`.

## `scripts/` — helper tooling

| File | Purpose |
|------|---------|
| `screenshots.mjs` | Capture README screenshots via Playwright |
| `screenshots-for-posts.mjs` | Capture screenshots at exact dimensions for external posts |
| `publish-post-screenshots.mjs` | Upload screenshots to WP media + swap URLs in posts |
| `insert-adminator-listings.mjs` | Insert Adminator entries into existing listicle posts |
| `move-adminator-position.mjs` | Reorder a listicle entry's position |

These exist for the maintainer's content workflow (publishing screenshots/listings to colorlib.com and adminlte.io). They aren't required for using the template.

## Where things live (cheat sheet)

| Want to… | Edit |
|----------|------|
| Change a color | `src/assets/styles/2026/_tokens.scss` |
| Change a font | `_tokens.scss` (or the `@import url(...)` line in `_base.scss`) |
| Add a sidebar nav item | `src/assets/scripts/2026/Shell.js` (the `NAV` array) |
| Add a new HTML page | Create `src/foo.html` + register in `webpack/plugins/htmlPlugin.js` + add to `Shell.js` NAV |
| Tweak a button / card / badge | `src/assets/styles/2026/_components.scss` (or `_ui.scss` for badges/alerts) |
| Add a new Chart.js chart | `src/assets/scripts/2026/charts.js` (the `SEEDS` object) |
| Change calendar events | `src/assets/scripts/2026/calendar.js` (the `SEED_EVENTS` array) |
| Change vector map markers | `src/assets/scripts/2026/maps.js` (the `MARKERS` array) |
| Add behavior on every page | `src/assets/scripts/2026/init.js` |
| Change build / port / chunks | `webpack/config.js` (entry, splitChunks) or `webpack/devServer.js` (port) |
