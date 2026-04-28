---
layout: default
title: Components
nav_order: 2
parent: Customization
---

# Component reference
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Every primitive in Adminator 4 is a custom CSS class — no Bootstrap utilities, no JS framework. The classes below cover everything used across the 18 pages.

For working examples of any of these, open <https://colorlib.com/polygon/adminator/ui.html>, <https://colorlib.com/polygon/adminator/forms.html>, or <https://colorlib.com/polygon/adminator/buttons.html>.

---

## Buttons

Defined in [`_components.scss`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/styles/2026/_components.scss) (base) and [`_ui.scss`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/styles/2026/_ui.scss) (variants).

```html
<!-- Solid -->
<button class="btn btn--primary">Primary</button>
<button class="btn btn--secondary">Secondary</button>
<button class="btn btn--success">Success</button>
<button class="btn btn--warning">Warning</button>
<button class="btn btn--danger">Danger</button>
<button class="btn btn--info">Info</button>
<button class="btn btn--ghost">Ghost</button>

<!-- Soft / tonal -->
<button class="btn btn--soft-primary">Primary</button>
<button class="btn btn--soft-success">Success</button>
<button class="btn btn--soft-warning">Warning</button>
<button class="btn btn--soft-danger">Danger</button>
<button class="btn btn--soft-info">Info</button>

<!-- Outline -->
<button class="btn btn--outline-primary">Primary</button>
<button class="btn btn--outline-success">Success</button>
<button class="btn btn--outline-danger">Danger</button>

<!-- Sizes -->
<button class="btn btn--primary btn--sm">Small</button>
<button class="btn btn--primary">Default</button>
<button class="btn btn--primary btn--lg">Large</button>

<!-- With icon -->
<button class="btn btn--primary">
  <svg viewBox="0 0 24 24"><path d="M12 5v14M5 12h14"/></svg>
  New
</button>

<!-- Icon-only -->
<button class="btn btn--ghost btn--icon" aria-label="Settings">
  <svg viewBox="0 0 24 24"><path d="..."/></svg>
</button>

<!-- Group / segmented control -->
<div class="btn-group">
  <button class="btn btn--ghost is-active">Day</button>
  <button class="btn btn--ghost">Week</button>
  <button class="btn btn--ghost">Month</button>
</div>
```

The `.btn` base sets typography + transitions. The `--variant` modifier sets colors. SVGs inside `.btn` get sized automatically (14×14 by default).

---

## Cards

```html
<section class="card">
  <div class="card-head">
    <div class="card-title-wrap">
      <span class="eyebrow">Section label</span>
      <h2 class="card-title">Card title</h2>
    </div>
    <a class="card-action" href="#">
      View report
      <svg viewBox="0 0 24 24"><path d="M5 12h14M13 5l7 7-7 7"/></svg>
    </a>
  </div>

  <!-- Card body content -->
</section>
```

Cards stack inside `.grid` (12-column) using `.col-6` (half-width) or `.col-12` (full):

```html
<div class="grid">
  <section class="col-6 card">…</section>
  <section class="col-6 card">…</section>
  <section class="col-12 card">…</section>
</div>
```

The grid collapses to 1 column under 720px.

---

## KPI cards (dashboard)

```html
<section class="kpi-grid">
  <article class="kpi-card c-success">
    <div class="kpi-top">
      <div class="kpi-identity">
        <div class="kpi-icon success">
          <svg viewBox="0 0 24 24"><path d="..."/></svg>
        </div>
        <div class="kpi-label">Total visits</div>
      </div>
      <span class="kpi-pill up">
        <svg viewBox="0 0 24 24"><path d="M7 17l10-10M7 7h10v10"/></svg>
        +10%
      </span>
    </div>
    <div class="kpi-value">1.24<sup>M</sup></div>
    <div class="kpi-compare">
      up from <strong>1.12M</strong> <span class="sep">·</span> last week
    </div>
  </article>
  <!-- 3 more kpi-cards… -->
</section>
```

`.c-success` controls the radial gradient color (the subtle glow in the corner). Available: `c-success`, `c-danger`, `c-purple`, `c-primary`.

`.kpi-icon.success`, `.kpi-icon.primary`, etc. control the small icon background.

`.kpi-pill.up`, `.kpi-pill.down`, `.kpi-pill.flat`, `.kpi-pill.info` control the trend pill colors.

---

## Forms

Defined in [`_forms.scss`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/styles/2026/_forms.scss).

### Field shell

```html
<div class="field">
  <label class="field-label" for="email">
    Email <span class="req">*</span>
  </label>
  <input id="email" class="input" type="email" placeholder="you@company.com">
  <div class="field-help">We'll never share your email.</div>
</div>
```

For an invalid state, add `is-invalid` and a `.field-error`:

```html
<input class="input is-invalid" value="not-an-email">
<div class="field-error">
  <svg viewBox="0 0 24 24"><circle cx="12" cy="12" r="10"/><path d="M12 8v5M12 16h.01"/></svg>
  Please enter a valid email address.
</div>
```

### Input with icon

```html
<div class="input-icon">
  <span class="ico"><svg viewBox="0 0 24 24"><circle cx="11" cy="11" r="7"/><path d="m21 21-4.3-4.3"/></svg></span>
  <input class="input" type="search" placeholder="Search...">
</div>
```

### Input group (addon prefix)

```html
<div class="input-group">
  <span class="addon">https://</span>
  <input class="input" type="text" placeholder="yourdomain.com">
</div>
```

### Select

```html
<select class="select">
  <option>Owner</option>
  <option selected>Admin</option>
  <option>Editor</option>
</select>
```

### Textarea

```html
<textarea class="textarea" placeholder="A few sentences..."></textarea>
```

### Checkbox

```html
<label class="check">
  <input type="checkbox" checked>
  <span class="box"></span>
  Subscribe to changelog
</label>
```

### Radio

```html
<label class="check radio">
  <input type="radio" name="plan" checked>
  <span class="box"></span>
  Starter — $0 / mo
</label>
```

### Toggle switch

```html
<label class="switch">
  <input type="checkbox" checked>
  <span class="track"></span>
  Enable two-factor auth
</label>
```

### Form layout grid

```html
<form>
  <div class="form-grid">
    <div class="field">
      <label class="field-label">First name</label>
      <input class="input" type="text">
    </div>
    <div class="field">
      <label class="field-label">Last name</label>
      <input class="input" type="text">
    </div>
    <div class="field span-2">
      <label class="field-label">Email</label>
      <input class="input" type="email">
    </div>
  </div>

  <div class="form-actions">
    <span class="badge dot success">All changes saved</span>
    <span class="spacer"></span>
    <button type="button" class="btn btn--ghost">Cancel</button>
    <button type="submit" class="btn btn--primary">Save</button>
  </div>
</form>
```

`.form-grid` is a 2-column responsive grid. Use `.span-2` to span both columns. Collapses to 1 column under 720px.

---

## Alerts

```html
<div class="alert success">
  <span class="ico">
    <svg viewBox="0 0 24 24"><path d="M22 11.1V12a10 10 0 1 1-5.9-9.1"/><path d="m22 4-10 10-3-3"/></svg>
  </span>
  <div class="body">
    <div class="title">All checks passed</div>
    CI completed in 1m 42s · 0 failing tests · ready to merge.
  </div>
  <button class="close" aria-label="Dismiss">
    <svg viewBox="0 0 24 24"><path d="M18 6 6 18M6 6l12 12"/></svg>
  </button>
</div>
```

Variants: `success`, `warning`, `danger`, `info`, `primary`. Each gets its own border color, background, and icon background.

---

## Badges

```html
<span class="badge">Default</span>
<span class="badge primary">Primary</span>
<span class="badge success">Success</span>
<span class="badge warning">Warning</span>
<span class="badge danger">Danger</span>
<span class="badge info">Info</span>
<span class="badge purple">Purple</span>
<span class="badge solid">Solid</span>

<!-- With status dot -->
<span class="badge success dot">Online</span>
<span class="badge warning dot">Idle</span>
<span class="badge danger dot">Down</span>
```

---

## Progress bars

```html
<!-- Default 8px -->
<div class="progress">
  <div class="progress-fill" style="width: 60%;"></div>
</div>

<!-- Variants -->
<div class="progress">
  <div class="progress-fill success" style="width: 100%;"></div>
</div>
<div class="progress">
  <div class="progress-fill warning striped" style="width: 82%;"></div>
</div>
<div class="progress">
  <div class="progress-fill gradient" style="width: 49%;"></div>
</div>

<!-- Sizes -->
<div class="progress thin">…</div>      <!-- 4px -->
<div class="progress tall">…</div>      <!-- 14px -->
```

Fill variants: `.success`, `.danger`, `.warning`, `.info`, `.gradient` (primary→purple), `.striped` (overlay pattern, combine with any color).

---

## Spinners

```html
<span class="spinner sm"></span>     <!-- 14px -->
<span class="spinner"></span>        <!-- 22px -->
<span class="spinner lg"></span>     <!-- 36px -->

<!-- Color override -->
<span class="spinner" style="border-top-color: var(--success);"></span>
```

---

## Tabs

### Underline tabs

```html
<div class="tabs">
  <a class="tab is-active" href="#">Overview</a>
  <a class="tab" href="#">Activity <span class="badge primary">12</span></a>
  <a class="tab" href="#">Settings</a>
</div>
```

### Pill tabs

```html
<div class="tabs pills">
  <a class="tab" href="#">Day</a>
  <a class="tab is-active" href="#">Week</a>
  <a class="tab" href="#">Month</a>
</div>
```

### With tab panels (data-driven, init.js wires it)

```html
<div data-tab-group>
  <div class="tabs">
    <a class="tab is-active" href="#" data-tab-target="a">A</a>
    <a class="tab" href="#" data-tab-target="b">B</a>
  </div>

  <div class="tab-panel is-active" data-tab-id="a">A content</div>
  <div class="tab-panel" data-tab-id="b">B content</div>
</div>
```

---

## Accordion

```html
<div class="accordion">
  <div class="accordion-item is-open" data-accordion>
    <button class="accordion-trigger" data-accordion-trigger>
      How does dark mode work?
      <span class="chev"><svg viewBox="0 0 24 24"><path d="m6 9 6 6 6-6"/></svg></span>
    </button>
    <div class="accordion-body">
      <div class="accordion-body-inner">
        Theme is stored in localStorage…
      </div>
    </div>
  </div>
  <!-- More accordion-item entries -->
</div>
```

`init.js`'s `initAccordions()` finds every `[data-accordion-trigger]` and wires the click handler.

---

## Modal (static demo)

```html
<div class="modal-demo">
  <div class="modal-head">
    <div class="modal-title">Delete project?</div>
    <button class="mail-tool" aria-label="Close">
      <svg viewBox="0 0 24 24"><path d="M18 6 6 18M6 6l12 12"/></svg>
    </button>
  </div>
  <div class="modal-body">
    Once deleted, the dashboard can't be recovered.
  </div>
  <div class="modal-foot">
    <button class="btn btn--ghost">Cancel</button>
    <button class="btn btn--danger">Delete</button>
  </div>
</div>
```

This is a styled card — there's no built-in show/hide JS. Wrap in your own backdrop + display logic when you need a real overlay modal.

---

## Tooltip / popover

```html
<span class="popover-demo">
  <span class="pop">Last sync 3 min ago</span>
  <button class="btn btn--ghost">Hover me</button>
</span>
```

The `.pop` is positioned absolutely above the parent and visible by default in this demo class. Wrap with hover styles to make it appear on hover.

---

## Dropdowns (header style)

These are the dropdowns in the topbar — notifications, messages, profile menu. Use the same pattern for any context menu:

```html
<div class="dd-wrap">
  <button class="icon-btn" data-dropdown aria-label="Menu">
    <svg viewBox="0 0 24 24"><path d="..."/></svg>
  </button>

  <div class="dd-menu" role="menu">
    <div class="dd-head">Section title</div>
    <div class="dd-list">
      <a class="dd-item" href="#">
        <div class="dd-avatar a1">JD</div>
        <div class="dd-body">
          <div class="dd-text"><strong>John Doe</strong> liked your <em>post</em></div>
          <div class="dd-time">5 MIN AGO</div>
        </div>
      </a>
      <!-- more dd-item entries -->
    </div>
    <a class="dd-footer" href="#">View all →</a>
  </div>
</div>
```

`init.js`'s `initDropdowns()` toggles `.is-open` on `.dd-wrap` when `[data-dropdown]` is clicked, closes others, and closes on outside-click and Escape.

Avatar gradient classes: `.a1` (primary→purple), `.a2` (success→teal), `.a3` (danger→warning).

For a profile-menu variant, add `.dd-profile`:

```html
<div class="dd-menu dd-profile" role="menu">
  <div class="dd-profile-head">
    <div class="dd-profile-name">John Doe</div>
    <div class="dd-profile-email">john@adminator.app</div>
  </div>
  <a class="dd-menu-item" href="#">
    <svg viewBox="0 0 24 24"><path d="..."/></svg>
    Settings
  </a>
  <div class="dd-divider"></div>
  <a class="dd-menu-item danger" href="#">
    <svg viewBox="0 0 24 24"><path d="..."/></svg>
    Logout
  </a>
</div>
```

---

## Tables

### Basic table

```html
<table class="table">
  <thead>
    <tr>
      <th>Name</th>
      <th>Status</th>
      <th>Date</th>
      <th style="text-align:right">Price</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td class="cell-name">Item #1</td>
      <td><span class="tag t-active">Active</span></td>
      <td class="cell-date">Apr 22</td>
      <td class="cell-price pos">$12</td>
    </tr>
  </tbody>
</table>
```

Tag variants: `t-new`, `t-used`, `t-old`, `t-unavail`, `t-active`, `t-info`.

### Data table (sortable + paginated)

For full-featured tables with sorting, search, filters, pagination — see the markup in [`src/datatable.html`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/datatable.html). The `.data-table` class + `.data-toolbar`, `.data-foot`, `.pager` form a complete pattern.

---

## Avatar group

```html
<div class="avatar-group">
  <span class="av ma-1">JD</span>
  <span class="av ma-2">SK</span>
  <span class="av ma-3">LR</span>
  <span class="av more">+8</span>
</div>
```

Avatar gradient classes (`ma-1` through `ma-6`): each is a different `linear-gradient(135deg, X, Y)` combination.

---

## Section heading helper

For "section labels" in dense pages:

```html
<h3 class="section-h">
  Notifications <span class="num">02</span>
</h3>
```

Used in forms.html and similar.

---

## Where to find the full source

Every component class is defined in one of these partials:

| Partial | Classes |
|---------|---------|
| [`_components.scss`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/styles/2026/_components.scss) | `.hero`, `.btn`, `.btn--ghost`, `.btn--primary`, `.card`, `.card-head`, `.grid`, `.col-*`, `.table`, `.tag` |
| [`_ui.scss`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/styles/2026/_ui.scss) | All button variants, alerts, badges, progress, spinner, tabs, accordion, modal, popover, avatar group |
| [`_forms.scss`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/styles/2026/_forms.scss) | `.field`, `.input`, `.select`, `.textarea`, `.check`, `.switch`, `.input-icon`, `.input-group`, `.form-grid`, `.form-actions` |
| [`_dropdowns.scss`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/styles/2026/_dropdowns.scss) | `.dd-wrap`, `.dd-menu`, `.dd-item`, `.dd-profile`, `.dd-menu-item` |
| [`_data.scss`](https://github.com/puikinsh/Adminator-admin-dashboard/blob/master/src/assets/styles/2026/_data.scss) | `.data-table`, `.data-toolbar`, `.data-foot`, `.pager` |

Page-specific partials (`_dashboard.scss`, `_email.scss`, `_calendar.scss`, `_chat.scss`, `_auth.scss`, `_error.scss`) define classes scoped to their respective pages — not generally reusable.
