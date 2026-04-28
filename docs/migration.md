---
layout: default
title: Migration from v3
nav_order: 8
permalink: /migration/
---

# Migration from v3
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Should you migrate?

Adminator v4 is a **rewrite, not an upgrade**. Class names, file paths, and JavaScript APIs all changed. The HTML markup of every page is different. The CSS uses none of v3's class names.

If you're starting fresh, use v4. If you have a v3 project with significant custom work, you have two options:

1. **Stay on v3.** The [`legacy-v3`](https://github.com/puikinsh/Adminator-admin-dashboard/tree/legacy-v3) branch is preserved indefinitely with security updates for at least 12 months. Pin via npm:

   ```bash
   npm install adminator-admin-dashboard@^3
   ```

2. **Adopt v4 as a new template.** Don't try to merge v4 into a v3 project — copy your custom content into the new shell instead. The rest of this guide covers what's where.

---

## What was removed

The entire `src/assets/scripts/` (except the new `2026/` folder) and `src/assets/styles/` (except `2026/`) was deleted. Specifically:

| v3 location | Status |
|-------------|--------|
| `src/assets/scripts/app.js` | Deleted |
| `src/assets/scripts/components/Sidebar.js` | Deleted |
| `src/assets/scripts/components/Chart.js` | Deleted |
| `src/assets/scripts/utils/{dom,events,storage,sanitize,logger,date,performance,theme}.js` | Deleted |
| `src/assets/scripts/{charts,chat,datatable,datepicker,email,fullcalendar,googleMaps,masonry,popover,scrollbar,search,skycons,ui,vectorMaps}/` | Deleted |
| `src/assets/styles/spec/` | Deleted |
| `src/assets/styles/vendor/` | Deleted |
| `src/assets/styles/utils/theme.css` | Deleted |
| `src/assets/styles/index.scss` | Deleted (replaced by `src/assets/styles/2026/index.scss`) |

Plus these npm dependencies:

- `bootstrap` + `@popperjs/core`
- `dayjs`
- `perfect-scrollbar`
- `masonry-layout`
- `load-google-maps-api`
- `skycons`
- `brand-colors`

---

## API mapping

If your custom v3 code calls any of these APIs, here's the v4 equivalent:

### Theme management

| v3 | v4 |
|----|----|
| `Theme.current()` | `document.documentElement.getAttribute('data-theme')` |
| `Theme.apply('dark')` | `document.documentElement.setAttribute('data-theme', 'dark')` |
| `Theme.toggle()` | Same as above (the toggle button click handler in `init.js` is the implementation) |
| Listening to `'adminator:themeChanged'` event | `MutationObserver` on `data-theme` attribute (see [JS API](api/theme-api#theme-change-observation)) |
| CSS variable `--c-bkg-card` | `--bg-card` (renamed) |
| CSS variable `--c-text-base` | `--t-base` (renamed) |
| CSS variable `--c-border` | `--border` (renamed) |
| CSS variable `--c-primary` | `--primary` (renamed) |

The naming convention changed: `--c-bkg-*` became `--bg-*`, `--c-text-*` became `--t-*`, `--c-border-*` became `--border-*`. The full new token list is in the [token system docs](customization/theme-system).

### Sidebar

| v3 | v4 |
|----|----|
| `new Sidebar(element)` | NAV manifest in [`Shell.js`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/scripts/2026/Shell.js) |
| `sidebar.render()` | `mountShell()` (called automatically on page load) |
| Sidebar markup in HTML | `<div data-shell-sidebar></div>` placeholder |
| `<a class="sidebar-link">` | `<a class="nav-link">` (rendered by Shell.js, not authored by hand) |

To add a sidebar item in v4, edit the `NAV` array in `Shell.js`. See [Page anatomy](architecture#shell-renderer).

### Charts

| v3 | v4 |
|----|----|
| `new ChartComponent(canvas, opts)` | `<canvas data-chart-key="my-chart">` + a seed in `SEEDS` |
| Per-instance theme listener | Automatic (one observer at module level) |
| Hardcoded colors in chart config | `t.primary`, `t.success`, etc. from the `tokens()` helper |
| Manual `Chart.defaults` setup | `applyDefaults(t)` runs once on init |

Adding a chart in v4 is two edits: the seed in `charts.js` and the `<canvas>` in your HTML. See [Practical recipes → Add a chart](examples/theme-integration#add-a-new-chartjs-chart).

### Calendar

| v3 | v4 |
|----|----|
| `new Calendar(...)` directly with manual theme handling | `initCalendarPage()` finds `[data-fc]` and wires it for you |
| Manual color overrides per event | `classNames: ['fc-cat-X']` + CSS in `_fullcalendar.scss` |

### Vector maps

| v3 | v4 |
|----|----|
| `new jsVectorMap(...)` directly | `initVectorMaps()` finds `[data-vmap]` and wires it |
| Inline color config | `MARKERS` array in `maps.js`, colors from CSS variables |

### DOM / events / storage utilities

The entire `utils/` module set is gone. v4 uses native browser APIs directly:

| v3 | v4 native equivalent |
|----|----------------------|
| `DOM.qs(selector)` | `document.querySelector(selector)` |
| `DOM.qsa(selector)` | `document.querySelectorAll(selector)` |
| `DOM.fadeIn(el)` | CSS transition + class toggle |
| `Events.delegate(parent, evt, sel, fn)` | `parent.addEventListener(evt, e => { if (e.target.matches(sel)) fn(e); })` |
| `Events.debounce(fn, n)` | Inline debounce or use lodash if you really need it |
| `Storage.get(key)` / `.set(key, val)` | `localStorage.getItem(key)` / `.setItem(key, val)` |
| `Sanitize.html(str)` | Use `textContent` instead of `innerHTML`, or DOMPurify if you must accept HTML |
| `Logger.debug(msg)` | `console.debug(msg)` (or strip in production via Webpack DefinePlugin) |
| `DateUtils.format(d)` (Day.js wrapper) | `Intl.DateTimeFormat` (native, no dependency) |

If your custom code relies heavily on these, the simplest migration is to copy the v3 utility files into your project (they were ~600 lines total) — but native APIs are the more future-proof path.

### App initialization

| v3 | v4 |
|----|----|
| `new AdminatorApp()` with component registry | `start()` in `index.js` calls `mountShell()` + 5 init functions |
| `app.register('charts', new ChartComponent())` | Just call your init function from `start()` |
| `app.init()` | Automatic on `DOMContentLoaded` |

---

## CSS class mapping

The biggest visual difference: v4 uses none of Bootstrap's classes. If you have custom HTML with Bootstrap utilities, here's the rough mapping:

| Bootstrap class | v4 equivalent |
|-----------------|---------------|
| `.btn .btn-primary` | `.btn .btn--primary` |
| `.btn .btn-outline-primary` | `.btn .btn--outline-primary` |
| `.btn-sm` / `.btn-lg` | `.btn--sm` / `.btn--lg` |
| `.card` | `.card` (similar but different padding/border) |
| `.alert .alert-success` | `.alert.success` |
| `.badge .badge-primary` | `.badge.primary` |
| `.dropdown` | `.dd-wrap` |
| `.form-control` | `.input` |
| `.form-select` | `.select` |
| `.form-check` | `.check` |
| `.row .col-md-6` | `.grid` with `.col-6` |
| `.text-success` / `.text-danger` | Use `var(--success)` / `var(--danger)` directly in your CSS |
| `.bg-primary` | Use `var(--primary)` directly |
| `.d-flex .align-items-center` | Inline `style="display: flex; align-items: center"` or write a class |
| Bootstrap modal markup | `.modal-demo` styled card + your own backdrop logic |

There's no utility-class system in v4 (no `.mt-3`, no `.text-center`). Component CSS uses semantic class names. If you want utility classes, add them to a new partial (e.g. `_utilities.scss`) — they're trivial to write.

---

## Custom theme migration

### v3 had

```css
:root {
  --c-bkg-body: #f4f6f9;
  --c-bkg-card: #ffffff;
  --c-text-base: #2a2a2a;
  --c-primary: #4d7cfe;
}
[data-theme="dark"] {
  --c-bkg-body: #15151d;
  /* …etc… */
}
```

### v4 has

```scss
:root[data-theme="light"] {
  --bg-body: #F0F4F8;
  --bg-card: #FFFFFF;
  --t-base: #1E293B;
  --primary: #2563EB;
  /* …28 more tokens, see _tokens.scss… */
}
:root[data-theme="dark"] {
  --bg-body: #0B1120;
  /* …matching dark variants… */
}
```

Substitutions: `--c-bkg-` → `--bg-`, `--c-text-` → `--t-`, `--c-border-` → `--border-`, `--c-primary` → `--primary`. Plus v4 added `--primary-soft` (for tinted alert/badge backgrounds), `--primary-ring` (for focus rings), `--*-soft` for every semantic color, and 3 shadow tokens.

If you have v3 brand colors you want to keep, just paste them into the new tokens with the new names:

```scss
:root[data-theme="light"] {
  --primary:       #4d7cfe;            /* old v3 brand color */
  --primary-light: #6E94FF;            /* lighter shade */
  --primary-dark:  #2D5BD9;            /* darker shade */
  --primary-soft:  #EDF2FF;            /* very light tint for backgrounds */
  --primary-ring:  rgba(77, 124, 254, 0.18);
}
```

---

## Page anatomy migration

v3 pages had Bootstrap-laden HTML throughout. v4 pages are much simpler — the shell is rendered by JS, so each page only contains its content area.

### v3 page structure

```html
<body class="app sidebar-mini">
  <!-- ~200 lines of sidebar markup -->
  <div class="sidebar">…</div>

  <!-- ~150 lines of header markup -->
  <div class="header">…</div>

  <!-- Page content -->
  <div class="container">
    <div class="row">
      <div class="col-md-12">
        <!-- Your content -->
      </div>
    </div>
  </div>
</body>
```

### v4 page structure

```html
<body data-active="my-page" data-crumbs="Section | My page">
  <div class="shell">
    <div data-shell-sidebar></div>     <!-- Shell.js fills in -->
    <div class="main">
      <div data-shell-topbar></div>    <!-- Shell.js fills in -->
      <main class="content">
        <!-- Your content -->
      </main>
      <div data-shell-footer></div>    <!-- Shell.js fills in -->
    </div>
  </div>
</body>
```

The chrome (sidebar, header, footer) is no longer in your HTML. To migrate a v3 custom page, throw away everything outside the content area and copy what's inside `<main class="content">` from a v4 page (e.g. [`blank.html`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/blank.html)) as your starting template. Then paste your v3 content where the placeholder content was.

---

## Build / npm scripts

The npm scripts kept the same names — `npm start`, `npm run build`, `npm run release:minified`, `npm run lint` all do what they did in v3. The output structure is similar (`dist/` folder with HTML files + bundled JS/CSS).

What changed:

- v3 had a single `main` entry. v4 has a single `2026` entry. Different filename, same idea.
- v3 emitted `main.js` (~4.5 MB minified). v4 emits `2026.js` (~24 KB minified). The reduction comes from dropping Bootstrap, jQuery, lodash, perfect-scrollbar, masonry, and the legacy components.
- v3 had a webpack-bundle-analyzer script. v4 still has it (`npm run build:analyze`).
- v3 had vitest + 3 test files. v4 has vitest in `devDependencies` but no tests yet (the v3 tests covered utility modules that no longer exist).

---

## Migration checklist

For each v3 project you want to bring to v4:

- [ ] Decide: stay on v3 (use `legacy-v3` branch / `^3` from npm) or rewrite on v4?
- [ ] Inventory custom theme colors → port to new token names in `_tokens.scss`
- [ ] Inventory custom HTML pages → restructure with v4 anatomy (`data-active`, `data-crumbs`, three placeholders, `<main class="content">`)
- [ ] Inventory custom JS → replace Theme/Sidebar/ChartComponent calls with the v4 equivalents above
- [ ] Inventory Bootstrap classes in your custom HTML → swap to v4 component classes
- [ ] Add custom pages to `webpack/plugins/htmlPlugin.js` and `Shell.js`'s NAV array
- [ ] Run `npm run lint` (must be 0/0)
- [ ] Run `npm run release:minified` (must succeed)
- [ ] Visually QA both light and dark themes on every custom page

---

## Want help?

If you have a v3 project and aren't sure how to migrate something specific, file an issue at <https://github.com/puikinsh/Adminator-admin-dashboard/issues> with a code snippet showing the v3 pattern. We'll add the equivalent to this guide.
