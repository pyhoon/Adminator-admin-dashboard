---
layout: default
title: API reference
nav_order: 5
has_children: true
permalink: /api/
---

# API reference

JavaScript surface area in Adminator 4.

There's no public framework here — the JS is intentionally small. Six modules in [`src/assets/scripts/2026/`](https://github.com/puikinsh/Adminator-admin-dashboard/tree/master/src/assets/scripts/2026), each focused on one job. Most pages don't need any custom JS at all.

If you're coming from v3, almost every API you used is gone. The legacy `AdminatorApp`, `Theme`, `Sidebar`, `ChartComponent`, and the `DOM` / `Events` / `Storage` / `Sanitize` / `Logger` utility modules were all removed. See the [migration guide](../migration) for v3 → v4 mapping.

## Pages in this section

- **[JavaScript API](theme-api)** — every module (`Shell.js`, `init.js`, `charts.js`, `calendar.js`, `maps.js`), what they export, and how to extend each
