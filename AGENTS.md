# Repository Guidelines

Firefly: an Astro 7 static blog (fork of Fuwari) with Svelte 5 islands and TypeScript. Chinese-first, i18n for en/zh_TW/ja/ko/ru. Deeper architecture notes live in `CLAUDE.md`; this file is the quick operational reference.

## Commands

pnpm only (`preinstall` enforces it). Node >=22 (CI uses 24).

- `pnpm dev` / `pnpm start` — dev server at `localhost:4321`.
- `pnpm check` — `astro check`.
- `pnpm type-check` — `tsc --noEmit --isolatedDeclarations` (covers `src/` and `scripts/`).
- `pnpm lint` / `pnpm format` — Biome over `./src ./scripts`; `lint` auto-fixes. CI only lints `./src`.
- `pnpm build` — full pipeline: `generate-github-card-data` -> `generate-lqips` -> `generate-vndb-covers` -> `astro build` -> `prune-pio-assets` -> `subset-fonts` -> `minify-inline-scripts` -> `run-pagefind`.
- `pnpm astro build` — raw Astro build only (what CI `build.yml` runs; skips LQIP/font/pagefind). Use for quick verification.
- `pnpm new-post <file>` / `pnpm new-dynamic` (alias `new-d`) — scaffold post / microblog entry.
- `pnpm lqips` / `pnpm github-cards` — regenerate committed data files.

## Architecture

- Config-driven: site behavior is toggled in `src/config/*.ts`, re-exported via `src/config/index.ts`; matching types in `src/types`. Edit config there, not hardcoded values in components.
- Path aliases (tsconfig): `@components/*`, `@assets/*`, `@constants/*`, `@utils/*`, `@i18n/*`, `@layouts/*` -> `src/<dir>/*`; `@/*` -> `src/*`.
- Content collections (`src/content.config.ts`): `posts`, `spec`, `dynamic`, `projects`. Note `projects` exists in code; some older docs omit it.
- `.astro` for static content/layouts; `.svelte` for interactive UI mounted with `client:load`/`client:visible`.
- Swup drives SPA page transitions; its container IDs are defined in `astro.config.mjs` (banner, swup, sidebars, floating TOC). Elements outside them won't update on navigation.
- Markdown is a unified remark/rehype pipeline in `astro.config.mjs`; repo-specific plugins live in `src/plugins/`.

## Performance constraints (do not regress)

`src/utils/fullscreen-wallpaper-utils.ts` and `src/utils/grid-layout-utils.ts` are rAF-throttled scroll paths tuned for mobile. Avoid per-frame `getComputedStyle`/layout reads and continuous full-screen `filter: blur()` writes. Full rationale is in `CLAUDE.md`.

## Generated & committed files

- `src/constants/lqips.json` (via `pnpm lqips`), `src/constants/icons-data.json`, `src/constants/github-card-data.json` are committed. The lqips and icons files are Biome-ignored.
- `dist/`, `public/vndb-covers/`, `.astro/` are gitignored. `Firefly-Docs/`, `Firefly-Lite/`, `package/` are excluded from the repo.
- Astro copies all of `public/`; `prune-pio-assets.ts` then strips unused 看板娘 assets from `dist/` based on pio config.

## Style

- Biome: tab indentation, double quotes, recommended rules; `useConst`/`useImportType`/`noUnusedVariables`/`noUnusedImports` disabled for `.svelte`/`.astro`/`.vue`.
- Component names `PascalCase`; config modules `camelCase` ending in `Config.ts`; utilities kebab-case.
- Conventional Commits (`feat:`, `fix:`, `chore:`). Keep PRs to one concern.

## CI & deployment

- `.github/workflows/`: `biome.yml` runs `biome ci ./src`; `build.yml` runs `pnpm astro check` then `pnpm astro build`; `deploy.yml` builds and publishes `dist/` to GitHub Pages on push to `master`.
- Other deploy targets: Vercel (`vercel.json`) and Cloudflare Workers (`CF_WORKERS` env + `wrangler.jsonc`).
