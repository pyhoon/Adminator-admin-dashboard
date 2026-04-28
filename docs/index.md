---
layout: default
title: Home
nav_order: 1
description: "Adminator 4 — vanilla-JS admin dashboard with token-driven CSS-variable design system"
permalink: /
---

# Adminator 4 Documentation
{: .fs-9 }

Vanilla-JS admin dashboard template. Token-driven CSS-variable design system. No jQuery. No Bootstrap. ~700 KB of production JS for all 18 pages.
{: .fs-6 .fw-300 }

[Get Started](getting-started/installation){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[Live Demo](https://colorlib.com/polygon/adminator/index.html){: .btn .fs-5 .mb-4 .mb-md-0 .mr-2 }
[GitHub](https://github.com/puikinsh/Adminator-admin-dashboard){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## What is Adminator 4

Adminator is a free, open-source admin dashboard template (MIT). The 4.0 release (April 2026) is a ground-up rewrite:

- **No framework CSS.** Bootstrap was removed. Every UI primitive — buttons, dropdowns, alerts, modals, tabs, accordions, progress bars, switches — is custom-built.
- **One source of truth for design.** Every color, font size, shadow, spacing token lives as a CSS variable in [`_tokens.scss`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/styles/2026/_tokens.scss). Light + dark variants are defined under `:root[data-theme="light"]` and `:root[data-theme="dark"]`.
- **One JS shell.** Sidebar, topbar, and footer render on every page from a single `NAV` manifest in [`Shell.js`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/scripts/2026/Shell.js). Adding a nav item is one line.
- **Real, themed library integrations.** Chart.js (6 chart types), FullCalendar (Day/Week/Month/Agenda), jsvectormap (world map). All read CSS variables on render and re-render automatically when you toggle the theme.

The legacy v3 codebase (Bootstrap-based) is preserved on the [`legacy-v3` branch](https://github.com/puikinsh/Adminator-admin-dashboard/tree/legacy-v3) and the [`v3.0.0` tag](https://github.com/puikinsh/Adminator-admin-dashboard/releases/tag/v3.0.0).

---

## Numbers

| Metric                     | v3.0.0           | v4.0.0           | Δ        |
| -------------------------- | ---------------- | ---------------- | -------- |
| Production JS (total)      | ~4.5 MB          | ~700 KB          | **−85%** |
| Production CSS             | ~280 KB          | 90 KB            | **−68%** |
| Top-level npm dependencies | 16               | 8                | **−50%** |

---

## Quick start

```bash
git clone https://github.com/puikinsh/Adminator-admin-dashboard.git adminator
cd adminator
npm install
npm start
```

Then open <http://localhost:4000>. See the [Installation](getting-started/installation) page for full prerequisites.

---

## Pages included

18 pages, all sharing the same shell. Auth and error pages skip the shell.

Dashboard · Email · Calendar · Chat · Compose · Charts · Forms · UI Elements · Buttons · Basic Table · Data Table · Google Maps · Vector Maps · Blank · Sign In · Sign Up · 404 · 500.

See [Pages reference](pages) for what each page contains.

---

## Where to start

**Setting up?** [Installation](getting-started/installation) → [Project structure](getting-started/project-structure) → [Development](getting-started/development).

**Need to understand the architecture?** [Page anatomy](architecture) shows how every page wires up to the shell.

**Customizing colors / brand?** [Token system](customization/theme-system) is the only file you should ever need to edit.

**Adding a new page?** [Adding a page](adding-a-page) — three small edits, no boilerplate.

**Coming from v3?** [Migration guide](migration) — what changed and how to move custom work over.
