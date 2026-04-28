---
layout: default
title: Architecture
nav_order: 3
permalink: /architecture/
---

# Architecture
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Page anatomy

Every page in Adminator 4 follows the same shape. Two `<body>` data attributes tell the shell what to render, three placeholder `<div>`s tell it where:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Page title · Adminator</title>

    <!-- Early-paint theme bootstrap.
         Sets data-theme BEFORE the browser paints, so dark-mode users
         never see a flash of light content. Must be inline. -->
    <script>
      (function () {
        try {
          var saved = localStorage.getItem('dash26-theme');
          var prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
          document.documentElement.setAttribute('data-theme', saved || (prefersDark ? 'dark' : 'light'));
        } catch (e) {
          document.documentElement.setAttribute('data-theme', 'light');
        }
      })();
    </script>
  </head>

  <body data-active="dashboard" data-crumbs="Workspace | Dashboard">
    <div class="shell">
      <div data-shell-sidebar></div>          <!-- ← Shell.js fills this -->
      <div class="main">
        <div data-shell-topbar></div>         <!-- ← Shell.js fills this -->
        <main class="content">
          <!-- Page-specific content goes here -->
        </main>
        <div data-shell-footer></div>        <!-- ← Shell.js fills this -->
      </div>
    </div>
  </body>
</html>
```

Webpack injects the CSS link and JS script into `<head>` automatically (see HtmlWebpackPlugin in `webpack/plugins/htmlPlugin.js`).

### `<body>` attributes

| Attribute | Required | Notes |
|-----------|---------|-------|
| `data-active` | yes (for shell pages) | Matches a `key` in `Shell.js`'s `NAV` manifest. Drives sidebar highlight. |
| `data-crumbs` | yes (for shell pages) | Pipe-separated. Last segment renders bold/colored as the current page. Example: `"Communications | Email | Inbox"`. |

### The three placeholders

| Placeholder | Replaced with |
|-------------|---------------|
| `<div data-shell-sidebar>` | Full sidebar (brand, nav sections, footer workspace card) |
| `<div data-shell-topbar>` | Topbar (breadcrumbs, search command, notifications, messages, theme toggle, profile) |
| `<div data-shell-footer>` | Footer (copyright + version metadata) |

If a placeholder is missing, that piece is silently skipped. Standalone pages (signin, signup, 404, 500) skip the shell entirely — no placeholders, no `data-active`/`data-crumbs`.

### What goes inside `<main class="content">`

Whatever you want — this is the page-specific area. Most pages start with a hero block:

```html
<main class="content">
  <section class="hero">
    <div class="hero-text">
      <span class="eyebrow" id="heroDate">Saturday · April 27 · 2026</span>
      <h1 class="hero-title">Welcome back, <span class="accent">John</span></h1>
      <p class="hero-sub">Subtitle text.</p>
    </div>
    <div class="hero-actions">
      <button class="btn btn--ghost">Export</button>
      <button class="btn btn--primary">New report</button>
    </div>
  </section>

  <!-- Then KPI grid, .grid with .card sections, etc. -->
</main>
```

The `id="heroDate"` is wired by `init.js` — if present, it gets today's date in "Day · Month DD · YYYY" format on every page load. Optional.

---

## Shell renderer

[`Shell.js`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/scripts/2026/Shell.js) holds the entire navigation as a single JavaScript object (`NAV`) and renders three pieces of chrome from it:

- **Sidebar** — every nav section with its items and badges
- **Topbar** — search command box, notifications dropdown, messages dropdown, theme toggle, profile dropdown
- **Footer** — copyright + version

The renderer is deliberately old-school string concatenation (no virtual DOM, no JSX). The output is set via `outerHTML` once, on page load. There's no incremental re-rendering — if you change the NAV, the change is picked up next page load.

### NAV structure

```js
const NAV = [
  {
    label: 'Workspace',                    // Section heading
    items: [
      {
        key: 'dashboard',                  // Matches data-active on body
        text: 'Dashboard',                 // Display text
        href: 'index.html',                // Link
        icon: '<path d="M3 12 12 3..."/>', // Inner SVG path
      },
      {
        key: 'pro',
        text: 'Go Pro',
        href: '#',
        badge: { kind: 'pro', text: 'PRO' },  // Optional badge
        icon: '<path d="M12 2 15 8..."/>',
      },
    ],
  },
  // …more sections…
];
```

Items with submenus look like:

```js
{
  key: 'tables',
  text: 'Tables',
  icon: '<rect x="3" y="4" width="18" height="16"…/>',
  children: [
    { key: 'basic-table', text: 'Basic Table', href: 'basic-table.html' },
    { key: 'datatable',   text: 'Data Table',  href: 'datatable.html' },
  ],
},
```

The parent renders as a collapsible group; the chevron rotates on click via `init.js`'s `initNavGroups()`.

### Badge kinds

Three predefined kinds, each with its own color:

| `kind` | Background | Use for |
|--------|-----------|---------|
| `pro` | Solid `--primary` (white text) | Premium features |
| `hot` | `--danger-soft` + `--danger` text | Trending / urgent |
| `new` | `--success-soft` + `--success` text | New features |

Add new kinds by extending the `.nav-badge.X` rules in `_shell.scss`.

---

## Boot sequence

What happens when a page loads:

```
1. Browser parses <head>
   ├── Inline script sets <html data-theme="..."> from localStorage / OS preference
   └── <link rel="stylesheet" href="style.css"> queued (production)
       OR style-loader injects styles (dev)

2. Browser parses <body>
   └── 18-page placeholders + page-specific <main class="content">
       are visible but un-styled until CSS arrives

3. CSS arrives → page renders with correct theme (no flash)

4. <script src="2026.js"> defers until DOMContentLoaded

5. start() runs:
   ├── mountShell()
   │   ├── Reads <body data-active>, data-crumbs
   │   ├── Renders sidebar/topbar/footer into the three placeholders
   │   └── Adds wp-image-XXX classes, srcset etc. handled by browser
   ├── initShellBehaviors()
   │   ├── Theme toggle click handler
   │   ├── Hero date population (if #heroDate exists)
   │   ├── Nav-group expand handlers
   │   ├── Dropdown click + outside-click handlers
   │   ├── Todo checkbox handlers
   │   ├── Accordion click handlers
   │   └── Tab group click handlers
   ├── initCharts()       // No-op if no <canvas data-chart-key>
   ├── initVectorMaps()   // No-op if no [data-vmap]
   └── initCalendarPage() // No-op if no [data-fc]
```

The `init*` functions are guarded — calling them on a page that doesn't use them is free.

---

## Why JS-rendered shell instead of static HTML?

Three reasons:

1. **Single source of truth.** With the shell in HTML, adding a sidebar item meant editing 18 HTML files. With it in JS, it's one entry in the `NAV` array.

2. **Type-of-active highlight is free.** When you add a new page, you set `data-active="foo"` on its `<body>` and the shell highlights the matching nav item automatically — no per-page CSS class to add.

3. **Future-proofing.** When the design needs an update (e.g. adding a "command palette" button to the topbar), it's one edit in `Shell.js` and every page picks it up on next load. No grep-and-replace across HTML files.

The trade-off: a brief moment between CSS arriving and JS running where the placeholders show as empty `<div>`s. In practice this is a few hundred milliseconds and isn't visible because CSS hides the unstyled placeholders. If you really want server-rendered HTML, run a small build step that calls `mountShell()` server-side and inlines the result — but that's overkill for an admin template.

---

## Removed v3 patterns

If you're reading old v3 docs / code, here's what no longer exists:

| v3 pattern | v4 equivalent |
|------------|---------------|
| `class AdminatorApp { ... }` with component registry | Page anatomy + `Shell.js` + the 5 `init*` functions |
| `class Sidebar { ... }` | `NAV` manifest in `Shell.js` |
| `class ChartComponent` with theme-aware redraw | `SEEDS` object in `charts.js` + `<canvas data-chart-key>` |
| `Theme` utility class | `data-theme` attribute + `MutationObserver` |
| `DOM` / `Events` / `Storage` / `Sanitize` / `Logger` utilities | Native APIs (`document.querySelector`, `localStorage`, etc.) |
| Custom event `'adminator:themeChanged'` | `MutationObserver` on `data-theme` |

The full migration mapping is in the [migration guide](migration).
