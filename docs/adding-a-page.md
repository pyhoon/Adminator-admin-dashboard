---
layout: default
title: Adding a page
nav_order: 9
permalink: /adding-a-page/
---

# Adding a new page

Three small edits, no boilerplate.

---

## 1. Create the HTML file

Make `src/team-chat.html` (any name — the slug becomes the URL):

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
              <p class="hero-sub">Page-specific content goes here.</p>
            </div>
          </section>

          <!-- More <section class="card"> blocks here -->

        </main>
        <div data-shell-footer></div>
      </div>
    </div>
  </body>
</html>
```

The simplest starting point is to copy [`src/blank.html`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/blank.html) and edit from there.

## 2. Register it in webpack

Open `webpack/plugins/htmlPlugin.js` and add to the `titles` map:

```js
const titles = {
  'index': 'Adminator · Dashboard',
  // …existing entries…
  'team-chat': 'Adminator · Team chat',     // ← add this
};
```

The key matches the filename (without `.html`). Webpack will build `dist/team-chat.html` next time it compiles.

## 3. Add it to the sidebar

Open `src/assets/scripts/2026/Shell.js` and add an entry to the appropriate section's `items` array:

```js
{
  label: 'Communications',
  items: [
    // …existing items…
    {
      key: 'team-chat',                                   // matches data-active on body
      text: 'Team chat',                                  // sidebar label
      href: 'team-chat.html',                             // link target
      badge: { kind: 'new', text: 'NEW' },                // optional
      icon: '<path d="M21 11.5a8.38 8.38 0 0 1-.9 3.8..."/>',  // SVG path
    },
  ],
},
```

Find an icon at any free SVG icon library — the inline path data is what goes in. The viewBox is fixed at `0 0 24 24` by the renderer.

## 4. Restart the dev server

```bash
# Ctrl+C then…
npm start
```

Webpack picks up new HTML templates only on restart (the template files are read once when `htmlPlugin.js` runs).

The new page is live at <http://localhost:4000/team-chat.html>, the sidebar shows the new item, and clicking it navigates correctly.

---

## What if it's a standalone page (no shell)?

Skip the placeholder `<div>`s and `data-active`/`data-crumbs` on the body. Use one of the standalone CSS shells (`.auth-shell`, `.error-shell`) or write your own layout inside `<body>`.

Example — a custom maintenance page:

```html
<body>
  <div class="error-shell">
    <div class="error-card">
      <span class="error-eyebrow">Status</span>
      <div class="error-code">503</div>
      <h1 class="error-title">Scheduled maintenance</h1>
      <p class="error-sub">We'll be back at 03:00 UTC.</p>
    </div>
  </div>
</body>
```

Still register in `webpack/plugins/htmlPlugin.js`, but leave it out of the sidebar NAV (no point linking to a maintenance page from your normal nav).

---

## Adding a submenu item

If your new item belongs under an existing parent (e.g. another table type under "Tables"), add it to the parent's `children` array:

```js
{
  key: 'tables',
  text: 'Tables',
  icon: '<rect x="3" y="4" width="18" height="16" rx="2"/>...',
  children: [
    { key: 'basic-table', text: 'Basic Table', href: 'basic-table.html' },
    { key: 'datatable',   text: 'Data Table',  href: 'datatable.html' },
    { key: 'pivot',       text: 'Pivot Table', href: 'pivot.html' },  // ← new
  ],
},
```

The parent group will auto-expand when any child has `data-active` matching its `key`.
