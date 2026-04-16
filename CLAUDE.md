# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

## Commands

```bash
npm run dev      # Start dev server (Turbopack, default)
npm run build    # Build for production (Turbopack, default)
npm run start    # Start production server
npm run lint     # Run ESLint
```

To use Webpack instead of Turbopack: `next dev --webpack` / `next build --webpack`.

## Architecture

This is a **Next.js 16** app using the **App Router** with TypeScript and Tailwind CSS v4.

- `app/layout.tsx` — root layout (required, wraps all pages)
- `app/page.tsx` — home route (`/`)
- `app/globals.css` — global styles with Tailwind v4 imports
- `public/` — static assets served from `/`

## Key Next.js 16 differences

- **Turbopack is the default** bundler for both `next dev` and `next build`. The `--turbopack` flag is no longer needed.
- **`next build` no longer runs ESLint automatically** — run `npm run lint` separately.
- **`next lint` is removed** — use the ESLint CLI directly (already wired in `package.json`).
- **Turbopack config** is now a top-level `turbopack` key in `next.config.ts`, not under `experimental`.
- **Sass tilde imports** (`~package`) are not supported in Turbopack — use bare package names instead.
- **`middleware`** convention is deprecated; use `proxy` instead.

Read `node_modules/next/dist/docs/` for full API reference before making changes.
