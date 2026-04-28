---
layout: default
title: Installation
nav_order: 1
parent: Getting started
---

# Installation
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Prerequisites

| Tool | Version | Notes |
|------|---------|-------|
| **Node.js** | 18.12+ (LTS recommended) | Tested through Node 24 |
| **npm** | 9+ (ships with Node) | Yarn / pnpm work too |
| **Git** | any recent version | Only for cloning |

Check your versions:

```bash
node --version   # >= v18.12.0
npm --version    # >= 9.0.0
```

If Node is older, use [nvm](https://github.com/nvm-sh/nvm) (`nvm install --lts`) or [Volta](https://volta.sh/) (`volta install node@lts`).

## Option 1 — Clone the repo (recommended)

```bash
git clone https://github.com/puikinsh/Adminator-admin-dashboard.git adminator
cd adminator
npm install
npm start
```

Open <http://localhost:4000> — the dashboard appears.

The dev server runs on port 4000 by default. To change it, edit `webpack/devServer.js`.

## Option 2 — npm package

```bash
npm install adminator-admin-dashboard
```

The published package includes both `src/` (source) and `dist/` (pre-built production output). Use `dist/` directly if you only need the static HTML/CSS/JS, or copy `src/` into your project to customize.

## Option 3 — Download the prebuilt zip

Each release attaches a built `dist.zip` to the GitHub release. Grab it from <https://github.com/puikinsh/Adminator-admin-dashboard/releases> for a static-hosting drop-in (no Node required).

## Want the v3 (Bootstrap-based) version?

The v3 codebase is preserved on the [`legacy-v3`](https://github.com/puikinsh/Adminator-admin-dashboard/tree/legacy-v3) branch and the [`v3.0.0`](https://github.com/puikinsh/Adminator-admin-dashboard/releases/tag/v3.0.0) tag, with security updates for at least 12 months.

```bash
# Clone v3 directly
git clone -b legacy-v3 https://github.com/puikinsh/Adminator-admin-dashboard.git adminator-v3

# Or pin v3 from npm
npm install adminator-admin-dashboard@^3
```

## Verify the install

After `npm install` finishes, you should see:

```text
added 1067 packages, and audited 1068 packages in Xs
```

Then `npm start` should print:

```text
[webpack-dev-server] Project is running at:
[webpack-dev-server] Loopback: http://localhost:4000/
webpack compiled successfully in ~3000 ms
```

If compilation succeeds and you see the dashboard at <http://localhost:4000>, you're set up. Continue to [Project structure](project-structure).

## Troubleshooting

**Port 4000 in use?** Either kill the process holding it (`lsof -ti :4000 | xargs kill`) or change the port in `webpack/devServer.js`.

**`sass` errors during install?** Make sure your Node is 18.12 or newer. Older Node versions can't build the latest sass binary.

**Permission errors on macOS / Linux?** Don't `sudo npm install` — fix npm's permissions instead. See [the npm docs on this](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally).

**Slow install on Windows?** Add the project folder to Windows Defender's exclusion list — antivirus scanning of `node_modules` is the usual culprit.
