# Adminator 4 Documentation

This directory is the source for the docs site at <https://puikinsh.github.io/Adminator-admin-dashboard/>. Built by GitHub Pages with Jekyll + the [`just-the-docs`](https://github.com/just-the-docs/just-the-docs) remote theme.

## Structure

```text
docs/
├── _config.yml                 # Jekyll + just-the-docs config
├── index.md                    # Home

├── getting-started.md          # Section index
├── getting-started/
│   ├── installation.md         # Clone, install, run
│   ├── project-structure.md    # The src/ tree
│   ├── development.md          # npm scripts, dev workflow
│   └── build-deployment.md     # Production build + hosting

├── architecture.md             # Page anatomy, Shell.js, boot sequence
├── adding-a-page.md            # Step-by-step recipe

├── customization.md            # Section index
├── customization/
│   ├── theme-system.md         # Token system, dark mode mechanics
│   ├── components.md           # All UI primitives (buttons, alerts, forms, etc.)
│   └── library-integrations.md # Chart.js / FullCalendar / jsvectormap

├── api.md                      # Section index
├── api/
│   └── theme-api.md            # JS API for Shell.js, init.js, charts.js, etc.

├── examples.md                 # Section index
├── examples/
│   └── theme-integration.md    # Practical recipes

├── pages.md                    # All 18 page descriptions
├── migration.md                # v3 → v4 migration guide
├── COMPONENT_GUIDE.md          # How to build new components

└── README.md                   # This file
```

## Editing the docs

Just edit the `.md` files. GitHub Pages rebuilds and re-publishes on every push to `master`. The build typically takes 30–60 seconds; check status at <https://github.com/puikinsh/Adminator-admin-dashboard/actions> if it doesn't appear.

### Local preview

If you want to preview locally before pushing (recommended for substantial changes):

```bash
cd docs
bundle install              # one time, installs Jekyll + theme dependencies
bundle exec jekyll serve
```

Then open <http://localhost:4000>. Live-reloads on save.

The `Gemfile` in this folder is intentionally minimal — it pins `github-pages` so the local Jekyll matches what GitHub builds.

### Markdownlint

The project's `.markdownlint.json` (in the repo root) silences a few rules that conflict with just-the-docs syntax (`MD022`, `MD025`, `MD033`, etc.). Edit that file if your IDE complains about something legitimate.

## Style guidance

- **Comprehensive but no fluff.** Every section should answer "what is this and how do I use it?" without marketing copy.
- **Inline source links.** Whenever you reference a file or function, link to its location on GitHub: `[Shell.js](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/scripts/2026/Shell.js)`.
- **Code over prose.** When you can show it in 10 lines of code, don't write 3 paragraphs.
- **Use tables for parallel concepts** (token name + light value + dark value + use; component class + variant + when-to-use).
- **Keep nav_order stable.** Reordering breaks bookmarks. Append new pages with the next available `nav_order`.

## Sub-section conventions

Every sub-section page (anything in `getting-started/`, `customization/`, `api/`, `examples/`) needs front matter that ties it to its parent:

```yaml
---
layout: default
title: Whatever
nav_order: 1
parent: Getting started     # must match parent page's `title:`
---
```

The parent page (`getting-started.md` etc.) sets `has_children: true`.
