---
layout: default
title: Token system
nav_order: 1
parent: Customization
---

# Token system
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Overview

Every visual decision in Adminator 4 is a CSS custom property defined in **one** file: [`src/assets/styles/2026/_tokens.scss`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/styles/2026/_tokens.scss).

There are 28 tokens, each defined twice — once under `:root[data-theme="light"]` and once under `:root[data-theme="dark"]`. Switching themes is a single attribute flip on `<html>`; no JavaScript touches color values.

```scss
:root[data-theme="light"] {
  --primary:       #2563EB;
  --bg-card:       #FFFFFF;
  --t-base:        #1E293B;
  /* …25 more… */
}

:root[data-theme="dark"] {
  --primary:       #60A5FA;
  --bg-card:       #141B2D;
  --t-base:        #F1F5F9;
  /* …same 25 more, dark variants… */
}
```

Components reference tokens via `var(--…)`:

```scss
.btn--primary {
  background: var(--primary);
  color: #fff;
  box-shadow: 0 1px 2px var(--primary-ring);
}
```

That's the entire system. No SCSS variables, no JS color tables, no theme switcher logic to keep in sync.

---

## Complete token reference

### Surfaces

| Token | Light | Dark | Used for |
|-------|-------|------|----------|
| `--bg-body` | `#F0F4F8` | `#0B1120` | Outer page background |
| `--bg-card` | `#FFFFFF` | `#141B2D` | Card / panel backgrounds |
| `--bg-sidebar` | `#FFFFFF` | `#141B2D` | Sidebar background (separate from card so you can theme independently) |
| `--bg-hover` | `#F8FAFC` | `#1C2438` | Hover state for nav items, list rows |
| `--bg-muted` | `#F1F5F9` | `#1A2237` | Secondary surfaces — chat bubbles, chip backgrounds, code blocks |

### Text

| Token | Light | Dark | Used for |
|-------|-------|------|----------|
| `--t-base` | `#1E293B` | `#F1F5F9` | Primary text |
| `--t-muted` | `#64748B` | `#94A3B8` | Secondary text — descriptions, sublabels |
| `--t-light` | `#94A3B8` | `#64748B` | Tertiary — placeholders, captions, axis labels |
| `--t-inverse` | `#FFFFFF` | `#0B1120` | Text that sits on a colored background |

### Borders

| Token | Light | Dark | Used for |
|-------|-------|------|----------|
| `--border` | `#E4E8EF` | `#222C42` | Card borders, strong dividers |
| `--border-soft` | `#EEF1F5` | `#1A2237` | Subtle dividers within cards |

### Brand

| Token | Light | Dark | Used for |
|-------|-------|------|----------|
| `--primary` | `#2563EB` | `#60A5FA` | Brand color — buttons, active states, links |
| `--primary-light` | `#3B82F6` | `#93C5FD` | Hover lift on solid primary |
| `--primary-dark` | `#1D4ED8` | `#3B82F6` | Pressed / hover-darkening |
| `--primary-soft` | `#EFF6FF` | `#0F2847` | Tinted background for active nav, alert backgrounds, badge fills |
| `--primary-ring` | `rgba(37,99,235,.18)` | `rgba(96,165,250,.24)` | Focus rings, button shadows |

### Semantic

Each semantic color follows the same `solid + soft` pattern. Solid is for borders/text/icons on a tinted background. Soft is the tinted background.

| Token | Light | Dark | Used for |
|-------|-------|------|----------|
| `--success` / `--success-soft` | `#10B981` / `#ECFDF5` | `#34D399` / `#0F2A20` | "OK" states, positive trends |
| `--warning` / `--warning-soft` | `#F59E0B` / `#FFFBEB` | `#FBBF24` / `#2B1F08` | "Caution" |
| `--danger` / `--danger-soft` | `#EF4444` / `#FEF2F2` | `#F87171` / `#2B1414` | Destructive, errors, negative trends |
| `--info` / `--info-soft` | `#0EA5E9` / `#F0F9FF` | `#38BDF8` / `#0D2232` | Informational |
| `--purple` / `--purple-soft` | `#8B5CF6` / `#F5F3FF` | `#A78BFA` / `#1E1A2C` | "Personal" category, accents |
| `--pink` / `--pink-soft` | `#EC4899` / `#FDF2F8` | `#F472B6` / `#2A1424` | Birthdays, soft accents |
| `--teal` / `--teal-soft` | `#14B8A6` / `#F0FDFA` | `#2DD4BF` / `#0E2826` | Reserved for future categories |
| `--orange` / `--orange-soft` | `#F97316` / `#FFF7ED` | `#FB923C` / `#2A1810` | Holiday/category accent |

### Shadows

| Token | Used for |
|-------|----------|
| `--shadow-sm` | Subtle card elevation, button base shadow |
| `--shadow-card` | Default card shadow |
| `--shadow-lg` | Hover lift, dropdown menus, modals |

Dark theme shadows are darker and have higher opacity (the same elevation looks weaker on dark backgrounds, so the values compensate).

### Misc

| Token | Used for |
|-------|----------|
| `--overlay` | Translucent backdrop for the sticky topbar (paired with `backdrop-filter: blur()`) |

---

## How dark mode works

Three pieces:

1. **CSS** — `_tokens.scss` defines two complete sets of tokens, scoped by attribute selector:

   ```scss
   :root[data-theme="light"] { /* … */ }
   :root[data-theme="dark"]  { /* … */ }
   ```

2. **Early-paint script** — every page has this in `<head>`, before any CSS arrives:

   ```html
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
   ```

   This sets the right theme **before** the browser paints, so users in dark mode never see a flash of light content.

3. **Toggle button** — wired up in [`init.js`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/scripts/2026/init.js) (`initThemeToggle`). Clicking it flips the attribute and persists the choice to `localStorage` under the `dash26-theme` key.

That's the whole system. There's no `Theme.toggle()` API, no `themechange` event, no observer registration. If you need to react to theme changes from your own code, watch the attribute:

```js
new MutationObserver((records) => {
  if (records.some((r) => r.attributeName === 'data-theme')) {
    // theme changed — re-render whatever needs to know
  }
}).observe(document.documentElement, { attributes: true });
```

This is exactly how `charts.js`, `calendar.js`, and `maps.js` stay in sync.

---

## Customizing

### Change the brand color

Edit `--primary` (and its companions) in both theme blocks of `_tokens.scss`:

```scss
:root[data-theme="light"] {
  --primary:       #DB2777;          /* pink instead of blue */
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

Save → the dev server rebuilds → every component on every page picks up the new color via `var(--primary)`. Buttons, active nav, focus rings, charts, calendar event highlights — everything.

### Add a new token

1. Define it under both theme blocks in `_tokens.scss`:

   ```scss
   :root[data-theme="light"] { --brand-accent: #06B6D4; }
   :root[data-theme="dark"]  { --brand-accent: #67E8F9; }
   ```

2. Use it in any partial:

   ```scss
   .my-thing { background: var(--brand-accent); }
   ```

3. If charts / maps need it at render time, add it to `tokens()` in [`charts.js`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/scripts/2026/charts.js#L15-L34) or [`maps.js`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/scripts/2026/maps.js#L24-L34):

   ```js
   function tokens() {
     const cs = getComputedStyle(document.documentElement);
     return {
       /* … */
       brandAccent: cs.getPropertyValue('--brand-accent').trim(),
     };
   }
   ```

### Force a theme on a specific page

Override `data-theme` in that page's `<html>`:

```html
<html lang="en" data-theme="dark">
```

You'll also want to remove or update the early-paint script in `<head>` so it doesn't flip the attribute back from localStorage.

### Disable dark mode entirely

Quick way: in `_tokens.scss`, drop the `[data-theme="..."]` selectors entirely and keep only the light values under `:root`. The toggle button stops doing anything but won't error.

Cleaner way: also delete `initThemeToggle` from `init.js` and remove the `<button id="themeToggle">` from `Shell.js`'s topbar render.

---

## Why this approach

The token system replaces v3's `Theme` JavaScript module + dual-stylesheet approach. The two main wins:

1. **No flash on load.** The early-paint script sets the attribute before any pixels render. v3 had a moment where light-mode users saw dark content (or vice versa) because the theme was applied after JS loaded.

2. **No JS-level color logic.** v3 had `Chart.defaults.borderColor = themeColors.border` etc. scattered across components. v4 has zero of that — every color comes from `var(--…)` and Chart.js / FullCalendar / jsvectormap re-read the variables when the theme attribute changes.

If you're migrating custom v3 components: anywhere you had `Theme.current()` or `Theme.apply()` or a `themechange` listener, replace with `getComputedStyle(document.documentElement).getPropertyValue('--your-token')` and a `MutationObserver` on `data-theme`. See the implementations of `tokens()` and the observer in [`charts.js`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/scripts/2026/charts.js) for a complete pattern.
