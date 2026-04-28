---
layout: default
title: Customization
nav_order: 4
has_children: true
permalink: /customization/
---

# Customization

How Adminator is themed and how to make it your own.

Adminator 4 has one design source of truth: a single SCSS file with ~28 CSS variables. Editing colors, fonts, spacing, or shadows means touching that file — nothing else. There's no JS color logic, no theme classes scattered across components, no separate dark stylesheet to keep in sync.

## Pages in this section

- **[Token system](theme-system)** — every CSS variable, what it controls, and how dark mode works
- **[Components](components)** — the UI primitives (buttons, alerts, badges, modals, tabs, etc.) with markup and class reference
- **[Library integrations](library-integrations)** — how Chart.js, FullCalendar, and jsvectormap stay in sync with the theme
