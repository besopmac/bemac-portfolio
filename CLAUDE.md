# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

## Project

`bemac-portfolio` — Bruno Campos' personal portfolio at bemac.io. Single-page site with anchor navigation, served from a self-managed VPS (not Vercel).

`CONTEXT.md` (gitignored, present locally) is the working brief: positioning, design tokens, content sections, infra notes, and the current build phase. Read it before making decisions about content, visual identity, or section ordering — and update it when a new decision is made.

## Commands

```bash
npm run dev      # next dev --turbopack (http://localhost:3000)
npm run build    # next build
npm run start    # next start (production server — used by PM2 on VPS)
npm run lint     # eslint (flat config, extends next/core-web-vitals + next/typescript)
npm run commit   # commitizen — ALWAYS use this instead of `git commit` for conventional-commit prompts
```

There is no test runner configured.

## Stack & Versions

- Next.js **16.2.6** with App Router and Turbopack — read `node_modules/next/dist/docs/` before writing Next-specific code; APIs and conventions have changed from prior major versions.
- React 19.2, TypeScript 5 (strict), Tailwind CSS v4 (PostCSS plugin, `@import "tailwindcss"` in `styles/globals.css`, theme tokens via `@theme inline` — **no `tailwind.config.ts`**, all theming lives in CSS).
- Node 22 (via nvm locally and in CI).
- Path alias: `@/*` → repo root.

## Branch & Deploy Flow

```
feature/* → PR → develop → PR → main → auto-deploy to VPS
```

**Pushing to `main` triggers a production deploy** via `.github/workflows/deploy.yml`: GitHub Actions builds, then SSHes into the Hostinger VPS, runs `git pull && npm ci && npm run build && pm2 restart bemac`. Treat `main` as production — do not push directly without intent to deploy.

App location on VPS: `/var/www/bemac-portfolio`. PM2 process name: `bemac` (port 3000), reverse-proxied by Nginx with Certbot SSL.

## Commit Conventions

- Husky + commitlint enforce Conventional Commits via the `commit-msg` hook.
- Husky installation is skipped in CI: the `prepare` script checks `process.env.CI` before running `husky`. Do not "fix" this by removing the guard — CI will fail.
- Project uses these types in addition to standard ones: `content` (MDX/copy changes). See `CONTEXT.md` for the full type list.

## Current Code State vs. Target Architecture

The repo is in an early build phase. `app/` currently contains only `layout.tsx`, `page.tsx` (still the create-next-app placeholder), and `favicon.ico`. The target structure described in `CONTEXT.md` (`components/sections`, `components/ui`, `content/*.mdx`, `lib/`) has not been created yet.

Design tokens live in `styles/globals.css`: dark-mode palette (background `#0e0e0e`, surface `#141414`, text `#f0ede6`, muted `#888888`, accent `#c9a96e`) declared as CSS vars on `:root` and exposed as Tailwind utilities via `@theme inline` (`bg-background`, `bg-surface`, `text-text`, `text-accent`, etc.). The Tailwind `--spacing` scale is set to `0.5rem` so the default step matches the 8pt grid from `CONTEXT.md` (`p-1` = 8px, `p-2` = 16px, …).

Fonts are not loaded yet — that is item 2 of the build order in `CONTEXT.md`. Until then, `font-sans`/`font-mono` fall back to Tailwind v4's defaults.
