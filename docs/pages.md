---
layout: default
title: Pages reference
nav_order: 7
permalink: /pages/
---

# Pages reference
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

All 18 pages share the same shell (sidebar + topbar + footer) except where noted. Each page is a static HTML file in [`src/`](https://github.com/puikinsh/Adminator-admin-dashboard/tree/master/src) with no template engine — what you see in the file is what gets served.

Live previews link to <https://preview.colorlib.com/theme/adminator/>.

---

## Workspace

### Dashboard — [index.html](https://preview.colorlib.com/theme/adminator/index.html)

The default landing page. Contains:

- **Hero** with "Welcome back, John" greeting and Export / New report buttons
- **4 KPI cards** — Total visits, Page views, Unique visitors, Bounce rate (each with trend pill, comparison line, and colored radial accent)
- **Site visits** card — 4 region columns with progress bars, plus 3 radial KPIs
- **Monthly stats** card — Real Chart.js line chart with theme-aware colors
- **Todo list** — Interactive checkboxes, badge variants
- **Sales report** — Table with 7 rows + status tags
- **Weather** card — Today's weather with 7-day forecast
- **Quick chat** — Bubble chat UI with composer

`data-active="dashboard"`. Demonstrates: KPIs, real Chart.js, todo interaction, table styling, chat bubbles.

---

## Communications

### Email — [email.html](https://preview.colorlib.com/theme/adminator/email.html)

3-pane inbox layout (folder rail / message list / reader pane).

- **Folder rail** — Inbox/Starred/Snoozed/Sent/Drafts/Spam/Trash with counts; color-dot labels (Work, Team, Finance, etc.); storage progress bar
- **Message list** — 10 sample threads with unread dots, sender, subject, preview, label tags, attachment indicators, star toggle; tabs (Primary / Updates / Promotions); search field
- **Reader pane** — Full message with avatar, time, action toolbar (archive/delete/snooze/label/nav), body with quote block, attachments grid, collapsed reply rows, inline reply bar

`data-active="email"`. Demonstrates: complex multi-pane layout, list styling, attachment cards, message threading.

### Compose — [compose.html](https://preview.colorlib.com/theme/adminator/compose.html)

Full email composer with:

- **From** sender display, **To** recipient chips with × buttons, **CC**/**BCC** toggles, **Subject** large input
- **Toolbar** with font select, bold/italic/underline, lists, link, code, quote, attach/image
- **Editable body** (`contenteditable="true"`) with sample formatted content
- **Attachments grid** with PDF/XLS thumbnails
- **Footer** with autosave status, schedule button, discard, send

`data-active="compose"`. Demonstrates: rich form layout, recipient chips, toolbar styling, contenteditable body.

### Calendar — [calendar.html](https://preview.colorlib.com/theme/adminator/calendar.html)

Real FullCalendar instance with 24 seed events.

- **Left rail**: Quick-add CTA, mini-calendar with event dots, "My calendars" colored checkboxes (Personal/Work/Team/Travel/Finance/Birthdays/Holidays), upcoming events list with date blocks
- **Main calendar**: Toolbar with month title, prev/today/next, Day/Week/Month/Agenda segmented control, filter + New event button; FullCalendar mount
- **Events** include single-day, all-day, and multi-day spans (Lisbon trip Apr 9–13)

`data-active="calendar"`. Demonstrates: FullCalendar wiring, event categories, sidebar layout, responsive collapse.

### Chat — [chat.html](https://preview.colorlib.com/theme/adminator/chat.html)

2-pane conversation UI.

- **Left rail**: 8 conversations with presence dots (online/away/offline), unread badges, search field
- **Right pane**: Active conversation header with status, full message thread with day separators, "typing…" indicator, calendar invite card, composer with toolbar (attach/emoji/mention) + send

`data-active="chat"`. Demonstrates: conversation list, message bubbles (mine vs theirs), typing animation, composer.

---

## Components

### Charts — [charts.html](https://preview.colorlib.com/theme/adminator/charts.html)

6 themed Chart.js examples:

- **Revenue YoY** — Line chart with two datasets (current vs prior year, dashed)
- **Channels** — Bar chart with multi-color bars
- **Devices** — Doughnut chart with right-positioned legend
- **Benchmark** — Radar chart comparing two series
- **MRR by plan** — Stacked bar chart, quarterly
- **Sessions trend** — Filled area chart, 30-day daily

All re-render on theme toggle. Each is wrapped in a `.chart-canvas-wrap` for responsive sizing.

`data-active="charts"`. Demonstrates: every Chart.js type the seeds support.

### Forms — [forms.html](https://preview.colorlib.com/theme/adminator/forms.html)

Working "Profile settings" form + state demos:

- **Profile form** — Name, email (with icon), role/timezone selects, website with addon, bio textarea, switches for notifications, save actions row
- **Input states** — Default, filled, disabled, with-icon, invalid (with `field-error`)
- **Select & textarea** demos
- **Checkboxes & radios** with disabled state
- **Toggle switches** with disabled state

`data-active="forms"`. Demonstrates: every form primitive.

### UI Elements — [ui.html](https://preview.colorlib.com/theme/adminator/ui.html)

The component gallery:

- **5 alerts** (primary, success, warning, danger, info) with title, body, dismiss button
- **Badges** — All variants + dot variants
- **Avatar group** — 6 avatars with overflow `+8`
- **Progress bars** — Various widths and variants (success, warning striped, danger, success, gradient, thin)
- **Spinners** — sm / default / lg, plus colored variants
- **Tabs** — Underline + pills
- **Accordion** — Working expand/collapse
- **Modal** — Static demo with delete-project copy
- **Tooltips** — 3 popover examples

`data-active="ui"`. Demonstrates: every UI primitive in one place.

### Buttons — [buttons.html](https://preview.colorlib.com/theme/adminator/buttons.html)

All button variants in one gallery:

- **Solid** — Primary, Secondary, Success, Warning, Danger, Info, Ghost
- **Soft / tonal** — Same color set
- **Outline** — Primary, Success, Danger
- **Sizes** — Small, default, large
- **Icons** — Leading, trailing, icon-only, settings-icon
- **Group** — Day/Week/Month/Year segmented + bold/italic/underline/link toolbar
- **States** — Disabled, loading

`data-active="buttons"`.

### Basic Table — [basic-table.html](https://preview.colorlib.com/theme/adminator/basic-table.html)

Two table examples:

- **Monthly orders** — 8 rows with order ID, customer, plan, status, date, amount; status tags + price coloring
- **Server status** — 5 rows with service name, region, latency, uptime progress bar, status badge

`data-active="basic-table"`.

### Data Table — [datatable.html](https://preview.colorlib.com/theme/adminator/datatable.html)

Full-featured data table (no DataTables library — custom CSS):

- **Toolbar** — Search input with icon, filter button with count badge, role/status filter selects, columns button
- **Table** — 15 user rows with checkbox selection, avatar + name + email cell, role badge, department, status (active/pending/inactive), ID (mono font), last-active time, actions (view/edit/more)
- **Footer** — "Showing 1–15 of 142", per-page selector, pager with current page highlighted

`data-active="datatable"`. Demonstrates: complex table, row selection, pagination, mono-font cells.

### Google Maps — [google-maps.html](https://preview.colorlib.com/theme/adminator/google-maps.html)

Iframe embed of Google Maps + supplementary content:

- **Map** — Centered on Riga (the project's HQ), 480px tall iframe
- **HQ card** — Address, hours, phone, email
- **Other locations** — Table of office locations with status badges

`data-active="google-maps"`. No API key required (iframe embed). To point elsewhere, change the iframe URL.

### Vector Maps — [vector-maps.html](https://preview.colorlib.com/theme/adminator/vector-maps.html)

Real jsvectormap world map + supporting content:

- **World map** — Themed via CSS variables, 10 city markers
- **Top cities** — 6 cities with progress bars and trend badges
- **Continent split** — 4 KPI cards (Europe, Americas, Asia-Pac, Other)

`data-active="vector-maps"`. Demonstrates: jsvectormap wiring + theme integration.

---

## Pages section

### Blank — [blank.html](https://preview.colorlib.com/theme/adminator/blank.html)

Starter page with hero + empty card. Useful when prototyping a new screen — copy the file, change `data-active`/`data-crumbs`, fill in `<main class="content">`.

### Sign In — [signin.html](https://preview.colorlib.com/theme/adminator/signin.html)

Standalone (no shell). Split-screen:

- **Left** — Gradient brand panel with logo, marketing copy, customer quote, footer meta
- **Right** — Sign-in form (email + password + keep-signed-in + submit) followed by social-auth buttons (Google / GitHub / Apple)

Layout collapses to single-column under 900px (brand panel hides). Demonstrates: split-screen auth pattern, social-auth styling.

### Sign Up — [signup.html](https://preview.colorlib.com/theme/adminator/signup.html)

Same shell as signin, with a longer form: full name, work email, workspace URL (with addon), password (with strength bar), terms checkbox.

### 404 — [404.html](https://preview.colorlib.com/theme/adminator/404.html)

Standalone error card with:

- Eyebrow "ERROR · NOT FOUND"
- Large gradient "404" code
- Title + descriptive paragraph
- Two CTAs (Back to dashboard / Go back)
- Meta footer with status code + ref

### 500 — [500.html](https://preview.colorlib.com/theme/adminator/500.html)

Same pattern as 404, with "Try again" CTA that reloads the page.

---

## Page count

18 pages total: 11 shell pages (Dashboard, Email, Calendar, Chat, Compose, Charts, Forms, UI, Buttons, Basic Table, Data Table, Google Maps, Vector Maps, Blank) + 4 standalone (Sign In, Sign Up, 404, 500).

If you don't need a particular page, delete its HTML file from `src/` and remove the entry from `webpack/plugins/htmlPlugin.js` and the NAV in `Shell.js`. The page won't be built.
