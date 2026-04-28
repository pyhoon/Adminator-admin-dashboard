---
layout: default
title: JavaScript API
nav_order: 1
parent: API reference
---

# JavaScript API
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Modules at a glance

All JS lives in [`src/assets/scripts/2026/`](https://github.com/puikinsh/Adminator-admin-dashboard/tree/master/src/assets/scripts/2026). Six small modules, ~36 KB unminified.

| Module | Purpose |
|--------|---------|
| `index.js` | Entry — imports SCSS, mounts shell, runs init |
| `Shell.js` | NAV manifest + `mountShell()` renderer |
| `init.js` | `initShellBehaviors()` — theme toggle, dropdowns, accordions, tabs, todos, nav-groups |
| `charts.js` | `initCharts()` — Chart.js seeds, theme-aware |
| `calendar.js` | `initCalendarPage()` — FullCalendar wired to the page toolbar |
| `maps.js` | `initVectorMaps()` — jsvectormap world map |

The entry function:

```js
// src/assets/scripts/2026/index.js
function start() {
  mountShell();
  initShellBehaviors();
  initCharts();
  initVectorMaps();
  initCalendarPage();
}
```

Each `init*` function is a no-op if the page doesn't have a corresponding mount point. Calling them all on every page is harmless.

---

## Shell.js

### `mountShell()`

Renders the sidebar, topbar, and footer into the page placeholders.

```js
import { mountShell } from './Shell.js';
mountShell();
```

Reads two attributes from `<body>` to know what to render:

```html
<body data-active="dashboard" data-crumbs="Workspace | Dashboard">
```

| Attribute | Type | Effect |
|-----------|------|--------|
| `data-active` | string | Matches a `key` in the `NAV` manifest. The matching nav item gets `is-active`. |
| `data-crumbs` | string | Pipe-separated breadcrumb. Last segment renders with the `current` class. |

And expects three placeholder divs to fill:

```html
<div class="shell">
  <div data-shell-sidebar></div>
  <div class="main">
    <div data-shell-topbar></div>
    <main class="content"> ...page content... </main>
    <div data-shell-footer></div>
  </div>
</div>
```

Each placeholder gets replaced with the rendered HTML. If a placeholder is missing, that piece is silently skipped (useful for pages like `signin.html` that don't want the shell).

### `NAV` manifest

The single source of truth for sidebar contents. Each entry in `Shell.js` looks like:

```js
const NAV = [
  {
    label: 'Workspace',
    items: [
      { key: 'dashboard', text: 'Dashboard', href: 'index.html',
        icon: '<path d="M3 12 12 3l9 9"/><path d="M5 10v10h14V10"/>' },
      { key: 'pro', text: 'Go Pro', href: '#',
        badge: { kind: 'pro', text: 'PRO' },
        icon: '<path d="M12 2 15 8l6.5 1-4.8 4.6L18 20l-6-3-6 3 1.3-6.4L2.5 9 9 8z"/>' },
    ],
  },
  // …more sections…
];
```

| Field | Type | Notes |
|-------|------|-------|
| `label` (section) | string | Section heading shown above the items |
| `items[].key` | string | Used by `data-active` to match. Must be unique across the whole NAV. |
| `items[].text` | string | Display label |
| `items[].href` | string | Link target |
| `items[].icon` | string | Inner HTML of the SVG icon (path data, no `<svg>` wrapper) |
| `items[].badge` | object? | `{ kind: 'pro' \| 'hot' \| 'new', text: string }` — small badge on the right |
| `items[].children` | array? | Submenu items. Parent renders as a collapsible group. |

To add a new sidebar item, add an entry to the appropriate section. No other file change required.

---

## init.js

### `initShellBehaviors()`

Wires up every interactive behavior in the shell. Idempotent — calling more than once is harmless because each binding checks for existence.

What it sets up, by category:

| Behavior | Trigger | Implementation |
|----------|---------|----------------|
| Theme toggle | `#themeToggle` (the moon/sun button in topbar) | Flips `data-theme` on `<html>`, persists to `localStorage['dash26-theme']` |
| Hero date | `#heroDate` (any element with this id) | Sets `textContent` to today's date in "Day · Month DD · YYYY" format |
| Sidebar nav-group expand | `[data-nav-toggle]` inside `[data-nav-group]` | Toggles `is-open` class on the parent group |
| Header dropdowns | `[data-dropdown]` inside `.dd-wrap` | Toggles `is-open` on the wrap; closes other open dropdowns; closes on outside-click and Escape |
| Todo checkbox | `.todo-check` | Toggles `is-done` on the parent `.todo-item` |
| Accordion expand | `[data-accordion-trigger]` inside `[data-accordion]` | Toggles `is-open` on the parent accordion item |
| Tab groups | `.tab` inside `[data-tab-group]` | Adds `is-active` to clicked tab; switches `.tab-panel` matching `data-tab-target` ↔ `data-tab-id` |

There's no public API beyond the single `initShellBehaviors()` function. To add a new global behavior, add a private `initX()` and call it from `initShellBehaviors()`.

---

## charts.js

### `initCharts()`

Finds every `<canvas data-chart-key="...">` on the page and renders the matching seed from the `SEEDS` object. Re-renders all charts whenever `<html data-theme>` changes.

```js
import { initCharts } from './charts.js';
initCharts();
```

### Adding a new chart

1. Add a seed function to the `SEEDS` object in `charts.js`:

   ```js
   const SEEDS = {
     // …existing seeds…

     'my-chart': (t) => ({
       type: 'line',
       data: {
         labels: ['Jan', 'Feb', 'Mar'],
         datasets: [{
           label: 'Revenue',
           data: [10, 25, 18],
           borderColor: t.primary,
           backgroundColor: `${t.primary}24`,
           tension: 0.4,
           fill: true,
         }],
       },
       options: {
         responsive: true, maintainAspectRatio: false,
         plugins: { legend: { display: false } },
         scales: {
           y: { grid: { color: t.soft }, ticks: { color: t.light } },
           x: { grid: { display: false }, ticks: { color: t.light } },
         },
       },
     }),
   };
   ```

2. Add a `<canvas>` to any page:

   ```html
   <div class="chart-canvas-wrap" style="height: 240px;">
     <canvas data-chart-key="my-chart"></canvas>
   </div>
   ```

The seed function receives a `tokens` object (`t`) with the resolved color values for the active theme. Use `t.primary`, `t.success`, etc. — never hex literals — so your chart respects the theme.

### `tokens` reference

The `tokens()` helper inside `charts.js` returns:

```js
{
  primary, success, danger, warning, info, purple, pink, orange, teal,
  text, muted, light, border, soft, bg
}
```

`text` / `muted` / `light` map to `--t-base` / `--t-muted` / `--t-light`. `bg` maps to `--bg-card`. `soft` is `--border-soft`. The rest are direct.

For partial transparency, use template literal concatenation:

```js
backgroundColor: `${t.primary}24`,   // 24 = ~14% alpha
borderColor:     t.primary,
```

Chart.js's `Chart.defaults` are also tweaked in `applyDefaults()` (font family, tooltip styles, legend positioning) — these apply to every chart automatically.

---

## calendar.js

### `initCalendarPage()`

If the page contains `<div data-fc>`, mounts a FullCalendar instance into it. Wires the page's existing toolbar buttons (`.cal-nav-btn`, `.cal-today-btn`, `.cal-view-tab`) to FullCalendar's `prev() / next() / today() / changeView()` methods. Re-renders on theme change.

```js
import { initCalendarPage } from './calendar.js';
initCalendarPage();
```

The mount point:

```html
<div data-fc style="flex: 1; min-height: 540px;"></div>
```

### Adding events

Edit `SEED_EVENTS` in `calendar.js`:

```js
const SEED_EVENTS = [
  { title: 'Q2 kickoff',       start: '2026-04-01T09:00', classNames: ['fc-cat-work'] },
  { title: '✈ Lisbon trip',    start: '2026-04-09', end: '2026-04-13',
    allDay: true, classNames: ['fc-cat-travel'] },
  // …
];
```

Use the standard [FullCalendar event object](https://fullcalendar.io/docs/event-object). The `classNames` field is what controls color — see the next section.

### Event color categories

Adminator defines 6 category classes in [`_fullcalendar.scss`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/styles/2026/_fullcalendar.scss). Use them via `classNames` on each event:

| Class | Color token |
|-------|-------------|
| `fc-cat-work` | `--primary` (blue) |
| `fc-cat-team` | `--success` (green) |
| `fc-cat-personal` | `--purple` |
| `fc-cat-travel` | `--info` (cyan) |
| `fc-cat-finance` | `--warning` (amber) |
| `fc-cat-birthday` | `--pink` |

To add a new category, add a CSS rule in `_fullcalendar.scss`:

```scss
.fc .fc-event.fc-cat-launch {
  background: var(--orange-soft);
  color: var(--orange);
  border-left-color: var(--orange);
}
```

Then reference it: `classNames: ['fc-cat-launch']`.

### Loading from a backend

Replace `SEED_EVENTS` with a fetch (note: keep classNames so colors apply):

```js
new Calendar(host, {
  // …other options…
  events: async (info, successCallback, failureCallback) => {
    try {
      const r = await fetch(`/api/events?from=${info.startStr}&to=${info.endStr}`);
      const events = await r.json();
      successCallback(events);
    } catch (e) {
      failureCallback(e);
    }
  },
});
```

---

## maps.js

### `initVectorMaps()`

Finds every `<div data-vmap>` on the page and renders a jsvectormap world map into it, themed via CSS variables. Re-renders on theme change.

```js
import { initVectorMaps } from './maps.js';
initVectorMaps();
```

The mount point:

```html
<div data-vmap style="height: 420px; width: 100%;"></div>
```

### Editing markers

Edit `MARKERS` in `maps.js`:

```js
const MARKERS = [
  { name: 'Riga',         coords: [56.95, 24.10] },
  { name: 'New York',     coords: [40.71, -74.00] },
  // …
];
```

Each marker is `{ name: string, coords: [lat, lng] }`. The map shows the name on hover via the labels plugin.

### Customizing colors

Marker fill uses `--primary`. Hover fill uses `--purple`. Region fill uses `--bg-muted`. To override, edit the `markerStyle` and `regionStyle` blocks in `maps.js`'s `buildOne()` function.

---

## Theme change observation

If you build a custom widget that needs to react to theme changes (re-read tokens, swap colors, etc.):

```js
const observer = new MutationObserver((records) => {
  if (records.some((r) => r.attributeName === 'data-theme')) {
    rebuildMyWidget();
  }
});
observer.observe(document.documentElement, { attributes: true });
```

This is the same pattern `charts.js`, `calendar.js`, and `maps.js` use. There's no shared event bus or pub/sub.

---

## Removed in v4

These v3 APIs no longer exist. If you have v3 code that calls them, see the [migration guide](../migration).

| v3 API | v4 equivalent |
|--------|---------------|
| `AdminatorApp` | None — page anatomy + `Shell.js` replace it |
| `Theme.current()` | `document.documentElement.getAttribute('data-theme')` |
| `Theme.apply(name)` | `document.documentElement.setAttribute('data-theme', name)` |
| `Theme.toggle()` | Same as above; `init.js`'s click handler is the implementation |
| `'adminator:themeChanged'` event | `MutationObserver` on `data-theme` |
| `Sidebar` class | NAV manifest in `Shell.js` |
| `ChartComponent` | `SEEDS` object in `charts.js` + `<canvas data-chart-key>` |
| `DOM` / `Events` / `Storage` / `Sanitize` / `Logger` utilities | None — use native DOM APIs directly |
