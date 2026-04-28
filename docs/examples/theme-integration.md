---
layout: default
title: Practical recipes
nav_order: 1
parent: Examples
---

# Practical recipes
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Add a new sidebar nav item

Edit the `NAV` array in `src/assets/scripts/2026/Shell.js`:

```js
{
  label: 'Communications',
  items: [
    // …existing items…
    { key: 'team-chat', text: 'Team chat', href: 'team-chat.html',
      badge: { kind: 'new', text: 'NEW' },
      icon: '<path d="M21 11.5a8.38 8.38 0 0 1-.9 3.8..."/>' },
  ],
},
```

Set `data-active="team-chat"` on `<body>` of the new page so the item highlights.

That's it — the sidebar renders the same way on all 18 pages, so the new item appears everywhere.

---

## Add a new HTML page

Three edits, no boilerplate:

### 1. Create the page

`src/team-chat.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Team chat · Adminator</title>
    <script>
      (function () {
        try {
          var saved = localStorage.getItem('dash26-theme');
          var prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
          document.documentElement.setAttribute('data-theme', saved || (prefersDark ? 'dark' : 'light'));
        } catch (e) { document.documentElement.setAttribute('data-theme', 'light'); }
      })();
    </script>
  </head>
  <body data-active="team-chat" data-crumbs="Communications | Team chat">
    <div class="shell">
      <div data-shell-sidebar></div>
      <div class="main">
        <div data-shell-topbar></div>
        <main class="content">
          <section class="hero">
            <div class="hero-text">
              <span class="eyebrow">Communications</span>
              <h1 class="hero-title">Team chat</h1>
              <p class="hero-sub">Your page-specific content goes here.</p>
            </div>
          </section>

          <section class="card">
            <!-- …whatever you want… -->
          </section>
        </main>
        <div data-shell-footer></div>
      </div>
    </div>
  </body>
</html>
```

### 2. Register it in webpack

`webpack/plugins/htmlPlugin.js` — add to the `titles` map:

```js
const titles = {
  'index': 'Adminator · Dashboard',
  // …existing entries…
  'team-chat': 'Adminator · Team chat',
};
```

### 3. Add to the sidebar nav

See "Add a new sidebar nav item" above.

Restart the dev server (`npm start`). The new page is at <http://localhost:4000/team-chat.html>.

---

## Override the brand color (light + dark)

Edit `src/assets/styles/2026/_tokens.scss`:

```scss
:root[data-theme="light"] {
  --primary:       #DB2777;          /* pink */
  --primary-light: #EC4899;
  --primary-dark:  #BE185D;
  --primary-soft:  #FDF2F8;
  --primary-ring:  rgba(219, 39, 119, 0.18);
}
:root[data-theme="dark"] {
  --primary:       #F472B6;
  --primary-light: #F9A8D4;
  --primary-dark:  #EC4899;
  --primary-soft:  #2A1424;
  --primary-ring:  rgba(244, 114, 182, 0.24);
}
```

Save → dev server rebuilds → every component updates: buttons, active sidebar item, focus rings, chart lines, calendar event highlights, the avatar gradient (which uses `linear-gradient(135deg, var(--primary), var(--purple))`).

---

## Add a new Chart.js chart

In `src/assets/scripts/2026/charts.js`, add a seed function:

```js
const SEEDS = {
  // …existing…

  'mrr-trend': (t) => ({
    type: 'line',
    data: {
      labels: ['Jan','Feb','Mar','Apr','May','Jun','Jul'],
      datasets: [{
        label: 'MRR',
        data: [42, 56, 50, 78, 88, 96, 110],
        borderColor: t.success,
        backgroundColor: `${t.success}24`,
        tension: 0.4,
        fill: true,
        pointRadius: 0,
        borderWidth: 2.5,
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

Then in any HTML page:

```html
<section class="card">
  <div class="card-head">
    <div class="card-title-wrap">
      <span class="eyebrow">Revenue</span>
      <h2 class="card-title">MRR</h2>
    </div>
  </div>
  <div class="chart-canvas-wrap" style="height: 240px;">
    <canvas data-chart-key="mrr-trend"></canvas>
  </div>
</section>
```

The chart renders on page load and re-renders on theme toggle. No extra JS needed.

---

## Add a calendar event from a backend

Replace the static `SEED_EVENTS` with a fetch in `src/assets/scripts/2026/calendar.js`:

```js
new Calendar(host, {
  plugins: [dayGridPlugin, timeGridPlugin, listPlugin, interactionPlugin],
  initialView: 'dayGridMonth',
  initialDate: new Date(),
  headerToolbar: false,
  events: async (info, success, failure) => {
    try {
      const r = await fetch(`/api/events?from=${info.startStr}&to=${info.endStr}`);
      const events = await r.json();
      // Map your backend's event shape to FullCalendar's
      success(events.map((e) => ({
        title: e.title,
        start: e.starts_at,
        end: e.ends_at,
        allDay: e.all_day,
        classNames: ['fc-cat-' + (e.category || 'work')],
      })));
    } catch (err) { failure(err); }
  },
});
```

The `fc-cat-*` classes (`fc-cat-work`, `fc-cat-team`, etc.) drive event colors — see [JS API → calendar.js](../api/theme-api#event-color-categories).

---

## Add a new alert variant

Alerts already support `success`, `warning`, `danger`, `info`, `primary` (see [`_ui.scss`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/styles/2026/_ui.scss)). To add a new one (e.g. `purple`):

```scss
/* in _ui.scss */
.alert.purple { background: var(--purple-soft); border-left-color: var(--purple); }
.alert.purple .ico { background: color-mix(in oklab, var(--purple) 14%, transparent); color: var(--purple); }
```

Use it:

```html
<div class="alert purple">
  <span class="ico"><svg viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/></svg></span>
  <div class="body">
    <div class="title">Beta feature</div>
    You're previewing an experimental dashboard layout.
  </div>
</div>
```

---

## React to theme changes from custom code

```js
// Run this once during page init:
const observer = new MutationObserver((records) => {
  if (records.some((r) => r.attributeName === 'data-theme')) {
    refreshMyWidget();
  }
});
observer.observe(document.documentElement, { attributes: true });

function refreshMyWidget() {
  const cs = getComputedStyle(document.documentElement);
  const primary = cs.getPropertyValue('--primary').trim();
  const bg = cs.getPropertyValue('--bg-card').trim();
  // re-render widget with these colors
}
```

This is exactly the pattern used in `charts.js`, `calendar.js`, and `maps.js`.

---

## Build a custom dashboard layout

Use the existing grid utility:

```html
<main class="content">
  <section class="hero">
    <div class="hero-text">
      <span class="eyebrow">Today</span>
      <h1 class="hero-title">My custom dashboard</h1>
      <p class="hero-sub">Subtitle text.</p>
    </div>
  </section>

  <!-- 4-column KPI row -->
  <section class="kpi-grid">
    <article class="kpi-card c-success">
      <div class="kpi-top">
        <div class="kpi-identity">
          <div class="kpi-icon success">
            <svg viewBox="0 0 24 24"><path d="..."/></svg>
          </div>
          <div class="kpi-label">Total revenue</div>
        </div>
        <span class="kpi-pill up">+12%</span>
      </div>
      <div class="kpi-value">$1.24<sup>M</sup></div>
      <div class="kpi-compare">
        up from <strong>$1.10M</strong> last month
      </div>
    </article>
    <!-- 3 more kpi-cards… -->
  </section>

  <!-- 12-column grid for cards -->
  <div class="grid">
    <section class="col-6 card">
      <div class="card-head">
        <div class="card-title-wrap">
          <span class="eyebrow">Performance</span>
          <h2 class="card-title">Monthly trend</h2>
        </div>
      </div>
      <div class="chart-canvas-wrap" style="height: 240px;">
        <canvas data-chart-key="my-trend"></canvas>
      </div>
    </section>

    <section class="col-6 card">
      <!-- … -->
    </section>
  </div>
</main>
```

Available column classes: `.col-6` (half-width), `.col-12` (full-width). The grid is 12 columns under the hood and collapses to 1 column under 720px.

---

## Embed a tab group

```html
<div data-tab-group>
  <div class="tabs">
    <a class="tab is-active" href="#" data-tab-target="overview">Overview</a>
    <a class="tab" href="#" data-tab-target="activity">Activity</a>
    <a class="tab" href="#" data-tab-target="settings">Settings</a>
  </div>

  <div class="tab-panel is-active" data-tab-id="overview">
    Overview content
  </div>
  <div class="tab-panel" data-tab-id="activity">
    Activity content
  </div>
  <div class="tab-panel" data-tab-id="settings">
    Settings content
  </div>
</div>
```

`init.js`'s `initTabGroups()` finds every `[data-tab-group]` and wires the click handlers automatically.
