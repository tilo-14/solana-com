# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Monorepo overview

Official Solana Foundation website monorepo: five Next.js 15 apps sharing
packages, orchestrated by Turborepo, deployed on Vercel with multi-project
rewrites.

| App | Package name | Port | Asset prefix | Content source |
|-----|-------------|------|--------------|----------------|
| Web | `solana-com` | 3000 | — | Builder.io CMS |
| Templates | `solana-templates` | 3001 | `/templates-assets` | GitHub API (`solana-foundation/templates`) |
| Media | `solana-com-media` | 3002 | `/media-assets` | Keystatic (Git-backed) |
| Docs | `solana-docs` | 3003 | `/docs-assets` | Fumadocs MDX in `content/` |
| Accelerate | `solana-accelerate` | 3004 | — | Hardcoded MDX + components |

Shared packages: `@workspace/ui` (Radix-based), `@solana-com/ui-chrome`
(Header/Footer/Theme), `@workspace/i18n` (next-intl, 20 locales),
`@workspace/config-eslint`, `@workspace/config-typescript`.

## Commands

```bash
pnpm install                              # Install (pnpm 10.15.1 required)
pnpm dev                                  # Dev all apps
pnpm dev --filter solana-com              # Web only
pnpm dev --filter solana-docs             # Docs only
pnpm dev --filter solana-com-media        # Media only (alias: pnpm dev:media)
pnpm dev --filter solana-templates        # Templates only
pnpm dev --filter solana-com-accelerate   # Accelerate only (alias: pnpm dev:acc)

pnpm build                                # Build all
pnpm test                                 # Jest tests
pnpm lint                                 # ESLint all workspaces
pnpm format:check                         # Prettier check
pnpm format:all                           # Prettier fix
pnpm clean                                # Remove node_modules, .next, .turbo, .source

# Per-app type checking
cd apps/<app> && pnpm check-types

# Turborepo generators (scaffold new packages)
pnpm turbo gen
```

## Architecture

### Multi-CMS strategy

Each app uses a different content backend:

- **Web**: Builder.io — dynamic pages and redirects managed externally
- **Docs**: Fumadocs — MDX files in `apps/docs/content/`, generates `.source/`
  (gitignored) via `fumadocs-mdx` postinstall. Uses CodeHike for syntax
  highlighting and Mermaid for diagrams.
- **Media**: Keystatic — Git-backed CMS, admin UI at `/keystatic`. Collections:
  Post, Podcast, Author, Category, Tag, CTA, Switchback, Link. Local dev uses
  filesystem; prod uses GitHub App OAuth.
- **Templates**: Fetches template metadata from GitHub API
- **Accelerate**: Static MDX event pages with Framer Motion animations

### Routing

All apps use `[locale]` dynamic routing for i18n (20 languages via next-intl).
Docs also has catch-all routes like `[...slug]` for nested MDX content.

### Build

Production builds require `NODE_OPTIONS=--max-old-space-size=8192`. Turbo
handles dependency ordering (`build` depends on `^build`). Apps with asset
prefixes serve static files through Vercel rewrites.

## Code conventions

- Named exports only — no default exports for components/functions
- React Server Components by default; `'use client'` only when interactive
- SVG handling: `*.inline.svg` → React component, `*.svg` → static asset
- Import shared components from `@workspace/*` packages, not relative paths
- Tailwind CSS for styling; SCSS for complex cases; Bootstrap 5 legacy in Web app
- Kebab-case directories, component per folder
- Functional/declarative patterns, early returns, guard clauses
- Boolean naming: `isLoading`, `hasError`

## Pre-commit hooks

Husky runs `lint-staged` (Prettier on changed `*.{js,jsx,ts,tsx,json,md,scss}`)
then `pnpm lint`. Never use `--no-verify`.

## Prettier

2-space indent, trailing commas, semicolons, 80 char width, LF line endings.
MDX files: no trailing commas. See `.prettierrc` for full config.

## Environment variables

Defined in `turbo.json` (36 vars). Key ones: `NEXT_PUBLIC_BUILDER_API_KEY`,
`KEYSTATIC_*`, `SENTRY_AUTH_TOKEN`, `YOUTUBE_API_KEY`, `SIMPLECAST_API_KEY`,
`INKEEP_API_KEY`, `DEVELOPER_CONTENT_API_KEY`.

## Payments content index

### Documentation (apps/docs/content/docs/en/payments/)

Core concepts:
- `index.mdx` — Payments landing page
- `how-payments-work.mdx` — Wallets, stablecoins, token accounts, fees, transactions
- `interacting-with-solana.mdx` — Network interaction guide
- `developer-tools.mdx` — SDKs and tools reference (JS, Rust, Python, Go, Java)
- `production-readiness.mdx` — Devnet to mainnet migration
- `agentic-payments.mdx` — AI agent micropayments via x402 protocol

Send payments (`send-payments/`):
- `index.mdx` — Overview
- `basic-payment.mdx` — First payment tutorial
- `payment-with-memo.mdx` — Reconciliation memos
- `verify-address.mdx` — Address verification
- `payment-processing/index.mdx` — Processing overview
- `payment-processing/batch-payments.mdx` — Bulk processing
- `payment-processing/fee-abstraction.mdx` — Fee sponsorship

Accept payments (`accept-payments/`):
- `index.mdx` — Overview
- `payment-button.mdx` — Button integration
- `solana-pay.mdx` — Solana Pay integration
- `indexing.mdx` — Payment listening/indexing
- `verification-tools.mdx` — Verification tools

Advanced (`advanced-payments/`):
- `deferred-execution.mdx` — Deferred execution patterns
- `spend-permissions.mdx` — Permission delegation

All content translated into 19 additional locales.

### Web app pages (apps/web/src/app/[locale]/)

| Route | Component | Data file |
|-------|-----------|-----------|
| `/solutions/institutional-payments/` | `solutions-institutional-payments.tsx` | `src/data/solutions/institutional-payments.ts` |
| `/solutions/payments-tooling/` | `solutions-payments-tooling.tsx` | `src/data/solutions/payments-tooling.ts` |
| `/solutions/commerce-tooling/` | `solutions-commerce-tooling.tsx` | `src/data/solutions/commerce-tooling.ts` |
| `/developers/payments/` | `developers-payments.tsx` | `src/data/developers/payments.js` |

Supporting component: `src/components/solutions/institutional-payments/CTACards.tsx`

### Docs app routing (apps/docs/src/app/[locale]/docs/payments/)

- `layout.tsx` — Payments section layout with InkeepChatButton
- `payments.tsx` — Page component and metadata utilities
- `page.tsx` — Index route
- `[...slug]/page.tsx` — Catch-all for nested pages

### Media/blog posts referencing payments

Case studies: `solana-pay-autonomous-case-study`, `case-study-tiplink`,
`case-study-boba-guys`, `case-study-helio`, `solana-pay-shopify`,
`paypal-pyusd-on-solana`, `solana-fireblocks-institutional-treasury-infrastructure`,
`only-possible-on-solana-decaf`, `case-study-gainforest`

## Sub-app documentation

Each app has its own `CLAUDE.md` with app-specific architecture details:
`apps/accelerate/CLAUDE.md`, `apps/web/CLAUDE.md`, `apps/docs/CLAUDE.md`,
`apps/media/CLAUDE.md`, `apps/templates/CLAUDE.md`.
