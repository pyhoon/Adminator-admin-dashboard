# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture (2026 redesign)

This is the **Adminator 2026** template — a vanilla-JS admin dashboard with a token-driven CSS-variable design system. There is **one** entry bundle, **one** stylesheet root, **one** shell renderer.

- No jQuery. No Bootstrap. No legacy admin.js / Sidebar component.
- Theme (light/dark) is a `data-theme` attribute on `<html>`, set by an early-paint script in each page and toggled at runtime via `init.js`.
- Sidebar, topbar, and footer are rendered by `Shell.js` from a single `NAV` manifest. Pages provide three placeholder divs (`data-shell-sidebar`, `data-shell-topbar`, `data-shell-footer`) plus `<body data-active="..." data-crumbs="...">`.
- Heavy widgets: real **Chart.js** (`charts.js`), real **FullCalendar** (`calendar.js`), real **jsvectormap** (`maps.js`). All three read CSS variables at render and re-render on theme toggle via a `MutationObserver` on `data-theme`.

## File layout

```text
src/
├── *.html                       # 18 pages — each ~500 lines, mostly content
├── assets/
│   ├── scripts/2026/            # The only JS
│   │   ├── index.js             # entry — imports SCSS, mounts shell, runs init
│   │   ├── Shell.js             # NAV manifest + sidebar/topbar/footer renderers
│   │   ├── init.js              # theme toggle, dropdowns, nav-groups, todos, accordions, tabs
│   │   ├── charts.js            # Chart.js seeds + tokens()
│   │   ├── calendar.js          # FullCalendar seed events + toolbar binding
│   │   └── maps.js              # jsvectormap world map
│   ├── styles/2026/             # The only SCSS
│   │   ├── index.scss           # entry — imports all partials below
│   │   ├── _tokens.scss         # CSS variables, light + dark
│   │   ├── _base.scss           # reset, body, .eyebrow, .mono
│   │   ├── _animations.scss     # rise-in / bar-in / fade-in / draw / spin
│   │   ├── _shell.scss          # .shell, sidebar, topbar, footer chrome
│   │   ├── _dropdowns.scss      # .dd-* (notifications, messages, profile)
│   │   ├── _components.scss     # .hero, .btn, .card, .grid, .table, .tag
│   │   ├── _forms.scss          # inputs, select, textarea, check, radio, switch
│   │   ├── _ui.scss             # alerts, badges, progress, spinner, tabs, accordion, modal
│   │   ├── _auth.scss           # signin/signup split-screen shell
│   │   ├── _error.scss          # 404 / 500 cards
│   │   ├── _chat.scss           # 2-pane chat layout
│   │   ├── _data.scss           # data-table, pager
│   │   ├── _charts.scss         # chart-canvas-wrap, legend
│   │   ├── _dashboard.scss      # KPIs, sv-* (site visits), todo, weather
│   │   ├── _email.scss          # 3-pane email layout
│   │   ├── _calendar.scss       # mini-cal, rail, upcoming list
│   │   ├── _fullcalendar.scss   # FullCalendar token overrides
│   │   └── _responsive.scss     # all media queries in one place
│   └── static/                  # fonts, images, icons (copy-pluginned to dist/)
```

The 2026 webpack entry produces: `runtime.js`, `2026.js`, plus auto-split `vendor-fullcalendar.js` / `vendor-chartjs.js` / `vendors.js`. Every page links the same chunks via HtmlWebpackPlugin.

## Page anatomy

Every shell page is structured the same way:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>...</title>
    <script>
      // Early-paint theme bootstrap — sets data-theme before CSS arrives,
      // so dark-mode users don't see a light flash.
      (function () {
        try {
          var saved = localStorage.getItem('dash26-theme');
          var prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
          document.documentElement.setAttribute('data-theme', saved || (prefersDark ? 'dark' : 'light'));
        } catch (e) { document.documentElement.setAttribute('data-theme', 'light'); }
      })();
    </script>
  </head>
  <body data-active="dashboard" data-crumbs="Workspace | Dashboard">
    <div class="shell">
      <div data-shell-sidebar></div>
      <div class="main">
        <div data-shell-topbar></div>
        <main class="content">
          <!-- Page-specific content here -->
        </main>
        <div data-shell-footer></div>
      </div>
    </div>
  </body>
</html>
```

`data-active` matches a `key` in `Shell.js`'s `NAV` manifest (e.g. `dashboard`, `email`, `calendar`, `charts`). `data-crumbs` is a `|`-separated list — last segment is highlighted as the current page.

Standalone pages (signin, signup, 404, 500) skip the `.shell` wrapper and use their own layout (`.auth-shell`, `.error-shell`).

## Adding a new page

1. Create `src/foo.html` with the body anatomy above.
2. Add `'foo': 'Adminator · Foo'` to the `titles` map in `webpack/plugins/htmlPlugin.js`.
3. Add a sidebar entry to `NAV` in `src/assets/scripts/2026/Shell.js`. Set `key: 'foo'`, `href: 'foo.html'`, and an inline SVG path for the icon.
4. Restart the dev server (the webpack config picks up new templates only on restart).

## Adding a new chart

Add a seed function to `SEEDS` in `src/assets/scripts/2026/charts.js`, keyed by a slug:

```js
'my-chart': (t) => ({ type: 'line', data: { ... }, options: { ... } }),
```

Then in any page: `<canvas data-chart-key="my-chart"></canvas>`. The `t` argument is a tokens object with the active theme's `primary`, `success`, `danger`, etc. — use `t.primary`, ``` `${t.primary}24` ``` for transparency, etc., never hex literals.

## Adding a new theme variable

1. Add it to both `:root[data-theme="light"]` and `:root[data-theme="dark"]` in `_tokens.scss`.
2. Use it via `var(--your-token)` anywhere.
3. If charts/maps need it, add it to `tokens()` in `charts.js` / `maps.js` so they pick it up at render.

## Commands

- `npm start` — dev server at <http://localhost:4000> (HMR enabled).
- `npm run build` — production build to `dist/`.
- `npm run release:minified` — production build with minification.
- `npm run lint` — ESLint + Stylelint, must be 0/0.
- `npm run build:analyze` — bundle analyzer report.
- `npm run clean` — wipe `dist/`.

There are currently no tests — Phase 5 cleanup deleted the legacy ones along with the legacy utils they tested. New tests for `init.js`, `Shell.js`, etc. would go in `tests/` if/when added.

## Conventions

- **CSS variables, never hex.** All colors live in `_tokens.scss`.
- **`is-active` / `is-open` / `is-done`** for state classes (BEM-ish modifiers).
- **`data-` attributes drive JS**, never `id`s except for `themeToggle` and `heroDate`.
- **Inline `<svg>` icons** (24×24 viewBox, `stroke-width: 1.75–2`, `fill: none`). Avoid icon-font dependencies.
- **`'Inter'` for body, `'Inter Tight'` for display, `'JetBrains Mono'` for numerics/eyebrows.** Loaded once via the `index.scss` `@import url(...)` at the top.
