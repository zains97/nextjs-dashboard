# AGENTS.md

Instructions for AI coding agents working in this repository.

## Project overview

This is the **Next.js App Router Dashboard** starter from the [Next.js Learn course](https://nextjs.org/learn/dashboard-app). It is an invoicing dashboard for a fictional company called Acme. The app uses the App Router, React Server Components, PostgreSQL, and NextAuth.

Course docs are the source of truth for features not yet implemented (dashboard routes, auth wiring, Server Actions, etc.).

## Tech stack

| Layer | Choice |
| --- | --- |
| Framework | Next.js 16 (App Router, Turbopack in dev) |
| UI | React 19, TypeScript 5.7, Tailwind CSS 3 |
| Database | PostgreSQL via [`postgres`](https://github.com/porsager/postgres) (tagged template queries) |
| Auth | NextAuth v5 (`next-auth@5.0.0-beta.25`) |
| Validation | Zod |
| Package manager | **pnpm** (not npm/yarn) |
| Icons | `@heroicons/react` |

## Commands

```bash
pnpm install          # install dependencies
pnpm dev              # dev server at http://localhost:3000 (Turbopack)
pnpm build            # production build
pnpm start            # run production server
```

**Database seed** (requires `POSTGRES_URL` in `.env.local`):

```bash
curl http://localhost:3000/seed
```

Copy `.env.example` to `.env.local` and fill in Postgres + auth values before running against a real database.

There is no dedicated lint or test script in `package.json` today. Verify changes with `pnpm build` and manual checks in the browser.

## Environment variables

Defined in `.env.example`:

- `POSTGRES_URL` — primary database connection string (used by `app/lib/data.ts` and `app/seed/route.ts`)
- `AUTH_SECRET` — generate with `openssl rand -base64 32`
- `AUTH_URL` — e.g. `http://localhost:3000/api/auth`

Never commit `.env`, `.env.local`, or other secret files.

## Directory layout

```
app/
  layout.tsx          # root layout
  page.tsx            # landing page (/)
  lib/
    data.ts           # async data-fetching functions (server-only)
    definitions.ts    # shared TypeScript types
    utils.ts          # formatCurrency, pagination, chart helpers
    placeholder-data.ts  # seed data (stable UUIDs — do not change IDs)
  ui/                 # presentational & client components
    dashboard/        # sidenav, cards, charts, nav links
    invoices/         # tables, forms, pagination, status badges
    customers/        # customer table
    global.css        # Tailwind directives
  seed/route.ts       # GET handler to create tables + seed data
  query/route.ts      # temporary dev query route (course exercise)
public/
  customers/          # customer avatar images
```

Path alias: `@/*` maps to the repo root (see `tsconfig.json`).

## Architecture conventions

### Server vs client components

- **Default to Server Components** — no `'use client'` unless the component needs hooks, browser APIs, or event handlers.
- Fetch data in Server Components or Server Actions; call functions from `app/lib/data.ts`.
- Add `'use client'` only at the top of files that need interactivity (e.g. `app/ui/search.tsx`).

### Data layer

- All database access lives in `app/lib/data.ts` using the shared `sql` client:

```typescript
const sql = postgres(process.env.POSTGRES_URL!, { ssl: 'require' });
```

- Use tagged template literals for queries (parameterized — never concatenate user input into SQL strings).
- Wrap fetchers in `try/catch`, log with `console.error`, and throw descriptive `Error` messages.
- Run independent queries in parallel with `Promise.all` when appropriate (see `fetchCardData`).
- Types for query results belong in `app/lib/definitions.ts`.

### Money and amounts

Invoice amounts are stored in the database **in cents** (integer). Use `formatCurrency` from `app/lib/utils.ts` for display. When loading an invoice for editing, divide by 100 (see `fetchInvoiceById`).

### UI and styling

- Tailwind utility classes; custom blues in `tailwind.config.ts` (`blue-400`, `blue-500`, `blue-600`).
- Reuse existing patterns: gray-50 form backgrounds, `peer` + absolute-positioned Heroicons in inputs, rounded-md borders.
- Shared button styles: `app/ui/button.tsx`.
- Loading placeholders: `app/ui/skeletons.tsx`.
- Import icons from `@heroicons/react/24/outline` (or `/20/solid` where already used).
- Prefer `next/link` for internal navigation and `next/image` for optimized images when adding assets.

### Forms and mutations

- Form UI lives under `app/ui/invoices/` and `app/ui/login-form.tsx`.
- Server Actions and Zod validation are added per the Learn course — follow existing form field `name` attributes when wiring actions.
- Use `use-debounce` for search inputs that update URL search params.

### Route handlers

- `app/seed/route.ts` — seeds DB via `GET`; uses `sql.begin` for transactional seeding.
- `app/query/route.ts` — course scratch file; safe to remove when finished.

## Coding style

- TypeScript strict mode; explicit types for props and data shapes.
- Named exports for utilities; default exports for React components (matches existing files).
- Prefer `async function` for data fetchers and page components that await data.
- Keep changes focused — match the file you are editing, do not refactor unrelated code.

**Import paths** — use the `@/` alias:

```typescript
import { fetchRevenue } from '@/app/lib/data';
import Search from '@/app/ui/search';
```

**Component example** (server component page):

```typescript
import { fetchCardData } from '@/app/lib/data';
import CardWrapper from '@/app/ui/dashboard/cards';

export default async function Page() {
  const cards = await fetchCardData();
  return <CardWrapper {...cards} />;
}
```

## What to do

- Use pnpm for all package operations.
- Follow Next.js App Router file conventions (`page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx` in route segments).
- Await `params` and `searchParams` in Next.js 15+ page components (they are Promises).
- Add new UI under `app/ui/` grouped by feature (`dashboard/`, `invoices/`, `customers/`).
- Add new types to `app/lib/definitions.ts` and new queries to `app/lib/data.ts`.

## What to ask before doing

- Schema or migration changes beyond what the seed route creates.
- Adding new dependencies not already in `package.json`.
- Deleting or renaming seed data UUIDs in `placeholder-data.ts` (they are referenced by foreign keys).
- Force-pushing or rewriting git history.

## What not to do

- Do not commit secrets or `.env*` files.
- Do not switch package managers (stay on pnpm).
- Do not introduce an ORM unless explicitly requested — this project uses raw SQL via `postgres`.
- Do not add artificial delays to data fetchers in production code (the commented delay in `fetchRevenue` is demo-only).
- Do not create markdown docs unless asked.
- Do not create git commits or pull requests unless the user explicitly requests them.

## Course context

This repo is built incrementally through the Learn chapters: styling, routing, data fetching, mutations, auth, and metadata. When implementing a chapter feature:

1. Check whether UI components already exist under `app/ui/`.
2. Reuse `app/lib/data.ts` fetchers before adding duplicates.
3. Align new routes with the course structure (`/dashboard`, `/dashboard/invoices`, `/login`, etc.).

Reference: [Dashboard App course](https://nextjs.org/learn/dashboard-app).
