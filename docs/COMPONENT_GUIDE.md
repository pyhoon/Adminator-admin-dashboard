---
layout: default
title: Component dev guide
nav_order: 10
---

# Component development guide
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

This guide is for people **adding new UI components** to Adminator 4 — not for using existing ones (that's covered in [Components reference](customization/components)).

The v3 era of "register a component class with `AdminatorApp`" is over. v4 components are CSS classes plus, optionally, a small init function in `init.js`.

---

## What "component" means in v4

There are three possible levels of component:

1. **Pure CSS** — A class (or set) in one of the SCSS partials. No JS. Render-time behavior only. Most components fall here: buttons, cards, alerts, badges, progress bars.

2. **CSS + light JS** — A class plus a behavior in `init.js`. The behavior is wired by attribute (`[data-X]`) so it self-discovers on every page. Examples: tab groups, accordions, dropdowns.

3. **CSS + dedicated JS module** — A new module file under `src/assets/scripts/2026/`. Used when the component wraps an external library or has its own non-trivial state. Current examples: `charts.js`, `calendar.js`, `maps.js`.

Pick the lowest level that meets your needs. Levels 1 and 2 are appropriate for 95% of cases.

---

## Recipe — pure CSS component

Goal: add a "stat-tile" — a small inline KPI display.

### 1. Decide where the styles live

Pick the right partial:

| Partial | When to use |
|---------|-------------|
| `_components.scss` | Generic primitive used across pages |
| `_ui.scss` | UI primitive (alert/badge/progress/spinner family) |
| `_forms.scss` | Anything form-input-shaped |
| `_dashboard.scss` | Dashboard-specific |
| `_email.scss` / `_calendar.scss` / `_chat.scss` | Page-specific |

For a generic stat tile usable anywhere, that's `_components.scss`.

### 2. Write the styles

```scss
/* in _components.scss */

.stat-tile {
  display: flex;
  align-items: baseline;
  gap: 8px;
  padding: 14px 16px;
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 10px;
}

.stat-tile-label {
  font-size: 11px;
  color: var(--t-light);
  font-family: 'JetBrains Mono', monospace;
  letter-spacing: 0.06em;
  text-transform: uppercase;
}

.stat-tile-value {
  font-family: 'Inter Tight', sans-serif;
  font-weight: 700;
  font-size: 18px;
  color: var(--t-base);
  letter-spacing: -0.02em;
}

.stat-tile-delta {
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  color: var(--t-muted);
  margin-left: auto;
}
.stat-tile-delta.up   { color: var(--success); }
.stat-tile-delta.down { color: var(--danger); }
```

### 3. Use it

```html
<div class="stat-tile">
  <span class="stat-tile-label">MRR</span>
  <span class="stat-tile-value">$84.2K</span>
  <span class="stat-tile-delta up">+12%</span>
</div>
```

### 4. Done

That's the whole component. Save the file, the dev server rebuilds. The tile works in light + dark mode automatically because every color is `var(--…)`.

---

## Recipe — CSS + light JS

Goal: add a "show password" eye toggle on password inputs.

### 1. Decide on a data attribute

Use a `data-X` attribute as the marker. Convention: `data-show-password` on the password input.

```html
<input class="input" type="password" data-show-password>
```

### 2. Add the toggle behavior

In `init.js`, add an init function and call it from `initShellBehaviors()`:

```js
// inside init.js

function initShowPassword() {
  document.querySelectorAll('input[data-show-password]').forEach((input) => {
    // Wrap the input in a relative container with a button
    const wrap = document.createElement('div');
    wrap.style.position = 'relative';
    input.parentNode.insertBefore(wrap, input);
    wrap.appendChild(input);

    const btn = document.createElement('button');
    btn.type = 'button';
    btn.setAttribute('aria-label', 'Show password');
    btn.style.cssText = 'position:absolute;right:10px;top:50%;transform:translateY(-50%);background:none;border:0;color:var(--t-muted);cursor:pointer;';
    btn.innerHTML = '<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/><circle cx="12" cy="12" r="3"/></svg>';
    wrap.appendChild(btn);

    btn.addEventListener('click', () => {
      input.type = input.type === 'password' ? 'text' : 'password';
    });
  });
}

export function initShellBehaviors() {
  initThemeToggle();
  initHeroDate();
  initNavGroups();
  initDropdowns();
  initTodos();
  initAccordions();
  initTabGroups();
  initShowPassword();   // ← add this
}
```

### 3. Use it

```html
<input class="input" type="password" data-show-password>
```

The toggle appears automatically on every page that has a `data-show-password` input.

### Patterns to follow

- **Always use `data-X` attributes**, never `id`s. Multiple instances on a page should work without conflict.
- **Always use `if (!el) return;` guards.** `init.js` runs on every page, but most pages don't have most components.
- **Don't store state on the DOM element.** Use closures or external Maps if you need per-element state.
- **Prefer one-shot listeners over delegation.** Adminator pages are static; rebinding on dynamic content isn't a concern. (If you do need delegation, attach to `document`.)
- **Don't block paint.** `init.js` runs after `DOMContentLoaded` — heavy work should be deferred via `requestIdleCallback` or `setTimeout(fn, 0)`.

---

## Recipe — dedicated module (heavy library or stateful)

Goal: integrate a new library (e.g. ECharts) with theme awareness.

### 1. Create the module

`src/assets/scripts/2026/echarts.js`:

```js
/**
 * ECharts wiring for Adminator.
 * Pages declare charts as <div data-echart="key"></div>.
 */

import * as echarts from 'echarts';

function tokens() {
  const cs = getComputedStyle(document.documentElement);
  return {
    primary: cs.getPropertyValue('--primary').trim(),
    success: cs.getPropertyValue('--success').trim(),
    danger:  cs.getPropertyValue('--danger').trim(),
    text:    cs.getPropertyValue('--t-base').trim(),
    muted:   cs.getPropertyValue('--t-muted').trim(),
    border:  cs.getPropertyValue('--border-soft').trim(),
    bg:      cs.getPropertyValue('--bg-card').trim(),
  };
}

const SEEDS = {
  'sales-trend': (t) => ({
    backgroundColor: 'transparent',
    textStyle: { color: t.muted, fontFamily: 'Inter, sans-serif' },
    xAxis: { type: 'category', data: ['Mon','Tue','Wed','Thu','Fri'], axisLine: { lineStyle: { color: t.border } } },
    yAxis: { type: 'value', splitLine: { lineStyle: { color: t.border } } },
    series: [{
      type: 'line',
      data: [120, 132, 101, 134, 90],
      lineStyle: { color: t.primary, width: 2.5 },
      areaStyle: { color: t.primary + '22' },
      smooth: true,
    }],
  }),
};

const instances = new Map();

function buildAll() {
  const t = tokens();
  document.querySelectorAll('[data-echart]').forEach((el) => {
    const key = el.getAttribute('data-echart');
    const seed = SEEDS[key];
    if (!seed) return;
    let chart = instances.get(el);
    if (!chart) {
      chart = echarts.init(el);
      instances.set(el, chart);
    }
    chart.setOption(seed(t), true);
  });
}

export function initECharts() {
  if (!document.querySelector('[data-echart]')) return;
  buildAll();
  new MutationObserver((records) => {
    if (records.some((r) => r.attributeName === 'data-theme')) buildAll();
  }).observe(document.documentElement, { attributes: true });
  // Re-render on resize
  window.addEventListener('resize', () => instances.forEach((c) => c.resize()));
}
```

### 2. Wire it up

In `index.js`:

```js
import { initECharts } from './echarts.js';

function start() {
  mountShell();
  initShellBehaviors();
  initCharts();
  initVectorMaps();
  initCalendarPage();
  initECharts();   // ← add this
}
```

### 3. Use it

```html
<div data-echart="sales-trend" style="width:100%;height:300px;"></div>
```

The pattern is identical to `charts.js` / `calendar.js` / `maps.js` — copy any of them as a template.

---

## Anti-patterns

Things v3 did that v4 deliberately doesn't:

| Anti-pattern | Why it's gone |
|--------------|---------------|
| `class MyComponent { constructor(el) { … } }` | Adds 30+ lines for what's usually a 5-line behavior. Use a function + `data-X` attribute. |
| Component registry / DI container | Adminator pages are static. There's no need for runtime composition. |
| Custom event bus (`adminator:themechange`) | `MutationObserver` on `data-theme` is built-in, free, and 4 lines. |
| Inline style strings injected via JS | Use SCSS — that's what stylesheets are for. JS-injected styles defeat the token system. |
| Polyfilled `Promise` / `fetch` | Browser support is "evergreen modern" — IE11 isn't on the support matrix. |
| jQuery's `$(el).each(...)` | `document.querySelectorAll(sel).forEach(fn)` does the same job natively. |
| Library wrappers that "abstract" Chart.js / FullCalendar | Use them directly. The libraries are already well-designed. |

---

## When to break the conventions

These conventions cover the typical case. If you have a genuinely complex stateful widget (e.g. a kanban board with drag-and-drop, undo, or undo, multi-user cursors), the "data-attribute + init function" pattern is too small. Build a real module with proper state management — but keep the integration with the shell minimal: it should still be initialized from `start()` and use the `tokens()` + MutationObserver pattern for theme awareness.

The point of the patterns is to keep simple things simple, not to forbid complexity.
