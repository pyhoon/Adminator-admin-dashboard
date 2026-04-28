---
layout: default
title: Library integrations
nav_order: 3
parent: Customization
---

# Library integrations
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Adminator 4 wires three external libraries — Chart.js, FullCalendar, jsvectormap — and themes them via CSS variables. All three re-render automatically when the user toggles the theme.

The pattern is the same in each:

```js
function tokens() {
  const cs = getComputedStyle(document.documentElement);
  return { primary: cs.getPropertyValue('--primary').trim(), /* … */ };
}

function build() {
  const t = tokens();
  // configure the library with t.primary, t.success, etc.
}

new MutationObserver((records) => {
  if (records.some((r) => r.attributeName === 'data-theme')) build();
}).observe(document.documentElement, { attributes: true });
```

If you build your own integration, follow the same recipe.

---

## Chart.js

Source: [`charts.js`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/scripts/2026/charts.js)

### What it does

- Reads CSS variables into a `tokens` object.
- Sets `Chart.defaults` (font family, tooltip styling, legend positioning) once.
- Finds every `<canvas data-chart-key="X">` on the page and renders the matching seed from `SEEDS[X]`.
- Re-renders all charts when `<html data-theme>` changes.

### `Chart.defaults` overrides

Set by `applyDefaults(t)`:

```js
Chart.defaults.font.family = "'Inter', system-ui, sans-serif";
Chart.defaults.font.size = 12;
Chart.defaults.color = t.muted;
Chart.defaults.borderColor = t.soft;
Chart.defaults.plugins.legend.position = 'bottom';
Chart.defaults.plugins.legend.labels.usePointStyle = true;
Chart.defaults.plugins.legend.labels.padding = 16;
Chart.defaults.plugins.legend.labels.boxWidth = 8;
Chart.defaults.plugins.legend.labels.boxHeight = 8;
Chart.defaults.plugins.tooltip.backgroundColor = t.text;
Chart.defaults.plugins.tooltip.titleColor = t.bg;
Chart.defaults.plugins.tooltip.bodyColor = t.bg;
Chart.defaults.plugins.tooltip.padding = 10;
Chart.defaults.plugins.tooltip.cornerRadius = 6;
Chart.defaults.plugins.tooltip.displayColors = false;
```

Override these only if your design language differs (e.g. you want top-positioned legends).

### Seeds

`SEEDS` is an object keyed by string. Each value is a function `(t) => chartConfig`:

```js
'revenue-line': (t) => ({
  type: 'line',
  data: { labels: [...], datasets: [{ borderColor: t.primary, ... }] },
  options: { /* … */ },
}),
```

Markup:

```html
<canvas data-chart-key="revenue-line"></canvas>
```

Wrap in `.chart-canvas-wrap` for sizing:

```html
<div class="chart-canvas-wrap" style="height: 240px;">
  <canvas data-chart-key="revenue-line"></canvas>
</div>
```

### Token reference for chart authors

| Token property | CSS variable |
|----------------|--------------|
| `t.primary` | `--primary` |
| `t.success` / `t.danger` / `t.warning` / `t.info` / `t.purple` / `t.pink` / `t.orange` / `t.teal` | matching semantic tokens |
| `t.text` | `--t-base` |
| `t.muted` | `--t-muted` |
| `t.light` | `--t-light` |
| `t.border` | `--border` |
| `t.soft` | `--border-soft` |
| `t.bg` | `--bg-card` |

For partial transparency, concatenate with template literals:

```js
backgroundColor: `${t.primary}24`,   // ~14% alpha
```

### Adding a new chart type

For a chart type that needs additional config (e.g. radar with custom angle lines), the seed function returns the full Chart.js config — anything Chart.js supports works. See the existing `sources-radar` and `mrr-stacked` seeds for non-trivial examples.

---

## FullCalendar

Source: [`calendar.js`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/scripts/2026/calendar.js) and [`_fullcalendar.scss`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/styles/2026/_fullcalendar.scss)

### What it does

- Mounts a FullCalendar instance into `<div data-fc>`.
- Wires the page's existing toolbar buttons (`.cal-nav-btn`, `.cal-today-btn`, `.cal-view-tab`) to FullCalendar's `prev() / next() / today() / changeView()`.
- Sets event colors via category classes (`fc-cat-work`, `fc-cat-team`, etc.).
- Re-renders on theme change.

### Plugins included

```js
import dayGridPlugin from '@fullcalendar/daygrid';
import timeGridPlugin from '@fullcalendar/timegrid';
import listPlugin from '@fullcalendar/list';
import interactionPlugin from '@fullcalendar/interaction';
```

That's enough for Day/Week/Month/Agenda views and basic interaction (click, drag). To add other plugins (resource view, scheduler, etc.), import them in `calendar.js` and add to the `plugins` array.

### CSS variable mapping

[`_fullcalendar.scss`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/styles/2026/_fullcalendar.scss) maps FullCalendar's own CSS variables to our tokens:

```scss
[data-fc] {
  --fc-border-color:     var(--border-soft);
  --fc-page-bg-color:    var(--bg-card);
  --fc-neutral-bg-color: var(--bg-muted);
  --fc-event-bg-color:   var(--primary);
  --fc-today-bg-color:   color-mix(in oklab, var(--primary) 10%, transparent);
  /* …more… */
}
```

This means everything FullCalendar renders — grid lines, today highlight, event chips, time labels — picks up the active theme automatically.

### Event color categories

Set via `classNames` on each event:

```js
{ title: 'Q2 kickoff', start: '2026-04-01T09:00', classNames: ['fc-cat-work'] }
```

Available categories (defined in `_fullcalendar.scss`):

| Class | Color |
|-------|-------|
| `fc-cat-work` | `--primary` (blue) |
| `fc-cat-team` | `--success` (green) |
| `fc-cat-personal` | `--purple` |
| `fc-cat-travel` | `--info` (cyan) |
| `fc-cat-finance` | `--warning` (amber) |
| `fc-cat-birthday` | `--pink` |

Each renders as a soft-tinted background with solid colored text and a left border in the same tone. To add a new one:

```scss
.fc .fc-event.fc-cat-launch {
  background: var(--orange-soft);
  color: var(--orange);
  border-left-color: var(--orange);
}
```

### Loading events from a backend

Replace `events: SEED_EVENTS` with a function:

```js
events: async (info, success, failure) => {
  try {
    const r = await fetch(`/api/events?from=${info.startStr}&to=${info.endStr}`);
    const events = await r.json();
    success(events.map(e => ({
      title:     e.title,
      start:     e.starts_at,
      end:       e.ends_at,
      allDay:    e.all_day,
      classNames: ['fc-cat-' + (e.category || 'work')],
    })));
  } catch (err) { failure(err); }
}
```

FullCalendar calls this every time the visible range changes (week navigation, view switch).

---

## jsvectormap

Source: [`maps.js`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/scripts/2026/maps.js)

### What it does

- Mounts a world vector map into `<div data-vmap>`.
- Themes regions via `--bg-muted`, hover via `--primary`, markers via `--primary` and `--purple`.
- Re-renders on theme change.

### Customizing markers

Edit the `MARKERS` array in `maps.js`:

```js
const MARKERS = [
  { name: 'Riga',         coords: [56.95, 24.10] },
  { name: 'New York',     coords: [40.71, -74.00] },
  { name: 'San Francisco',coords: [37.77, -122.42] },
  // …
];
```

Coordinates are `[lat, lng]` — note the order (latitude first, the opposite of GeoJSON).

### Customizing colors

The `buildOne()` function builds the jsvectormap config. Find these blocks to adjust:

```js
regionStyle: {
  initial: { fill: t.soft, stroke: t.border, strokeWidth: 0.4, fillOpacity: 1 },
  hover:   { fill: t.primary, fillOpacity: 0.5 },
},
markerStyle: {
  initial: { fill: t.primary, stroke: t.bg, strokeWidth: 2, r: 5 },
  hover:   { fill: t.purple,  stroke: t.bg, strokeWidth: 2, r: 7 },
},
```

### Switching to a different region map

The world map is loaded from `'jsvectormap/dist/maps/world.js'`. To use US states or a single country, change the import and the `map: 'world'` config:

```js
import 'jsvectormap/dist/maps/us-aea.js';   // US Albers Equal Area
// …
new jsVectorMap({
  selector: host,
  map: 'us_aea',  // matches the map name from the imported file
  // …
});
```

See [jsvectormap's docs](https://github.com/themustafaomar/jsvectormap) for the full list of available maps.

---

## Adding a new themed library

If you wire up a new library (e.g. ApexCharts, Leaflet, ECharts), follow the same recipe:

```js
// my-widget.js
import { MyLibrary } from 'my-library';

function tokens() {
  const cs = getComputedStyle(document.documentElement);
  return {
    primary: cs.getPropertyValue('--primary').trim(),
    text:    cs.getPropertyValue('--t-base').trim(),
    bg:      cs.getPropertyValue('--bg-card').trim(),
    border:  cs.getPropertyValue('--border').trim(),
  };
}

let instance = null;

function build() {
  if (instance) instance.destroy();
  const t = tokens();
  instance = new MyLibrary({
    /* ...config using t.primary, etc... */
  });
}

export function initMyWidget() {
  if (!document.querySelector('[data-mywidget]')) return;
  build();
  new MutationObserver((records) => {
    if (records.some((r) => r.attributeName === 'data-theme')) build();
  }).observe(document.documentElement, { attributes: true });
}
```

Then in `index.js`:

```js
import { initMyWidget } from './my-widget.js';

function start() {
  mountShell();
  initShellBehaviors();
  initCharts();
  initVectorMaps();
  initCalendarPage();
  initMyWidget();
}
```

The `if (!document.querySelector('[data-mywidget]')) return` guard means calling it on every page is free — it only does work when the mount point exists.
