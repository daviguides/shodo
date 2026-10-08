# TypeScript Library Preferences Specification

TypeScript equivalents of the shodo Python stack, for frontend work —
chosen for speed (native first), bundle size, and type inference
(state of the ecosystem: Oct 2026).

**Existing projects keep their libraries.** These preferences guide
new projects and new dependencies only; replacing an installed library
is a project decision, never a standards side effect.

## Python → TypeScript Mapping

| Python (shodo) | TypeScript | Notes |
|----------------|-----------|-------|
| uv | **bun install** / **pnpm 12** | Native package managers — see style spec |
| ruff | **oxlint** + **oxfmt** | Rust lint + format |
| pydantic | **Valibot** / **ArkType** / **Zod 4** | Standard Schema; pick per constraint below |
| httpx | **fetch** | Built-in; wrap once in a typed client |
| typer | **citty** | See typescript-cli-architecture-spec |
| rich | **picocolors** | CLI only; frontend has no equivalent need |
| polars | **nodejs-polars** | Same Rust engine; data tooling only |
| fastapi | **Hono** | Only when a frontend project owns a BFF |
| pytest | **Vitest 4** | See typescript-testing-tools-spec |

## Frontend Framework
- **React 19** — function components, Actions, `use`, React Compiler
  where the bundler supports it
- Framework (Next.js, TanStack Start, React Router framework mode)
  only when SSR/routing needs justify it; a client SPA or federated
  remote stays a plain Vite/Rsbuild app

## Styling
- **Tailwind CSS v4** — Oxide engine (Rust) + Lightning CSS
- Component kit is a project choice (HeroUI, shadcn/ui, Radix) —
  follow what the project uses
- **No runtime CSS-in-JS** (styled-components, Emotion) — runtime
  style injection costs every render; zero-runtime only

## Data Validation
All three implement **Standard Schema** — integrations
(forms, routers, actions) accept any of them.

| Constraint | Choice |
|------------|--------|
| Bundle size matters (browser code) | **Valibot** — tree-shakes to the validators used |
| Raw parse throughput (hot paths, large payloads) | **ArkType**, or **Typia** when an AOT transform is acceptable |
| Ecosystem/integration breadth | **Zod 4** (`zod/mini` in the browser) |

- Schema is the source of truth: derive the type
  (`v.InferOutput<typeof Schema>`), never write both by hand
- Parse at the boundary (fetch response, storage, URL, postMessage),
  trust inside

## State and Data Fetching
- **Local state first** — `useState`/`useReducer`; lift only when two
  siblings need it
- **Server state: TanStack Query** — caching, dedup, retries; never
  hand-rolled `useEffect` + `fetch` caches
- **Shared client state: Zustand** when context re-render cost or
  prop depth hurts; no Redux in new projects
- URL is state: filters, tabs, pagination live in search params

## Routing
- **TanStack Router** or **React Router 7** — typed params and search;
  follow the project's existing router

## Forms
- **React 19 Actions** for simple forms; **TanStack Form** or
  **React Hook Form** with a Standard Schema resolver for complex ones

## Module Federation
- **Rsbuild 2 + `@module-federation/rsbuild-plugin`** (MF 2.0) —
  the federation stack; Vite federation plugins lag in runtime
  features and type sharing
- Share framework libraries as **singletons** (`react`, `react-dom`,
  design system, realtime client) — two Reacts on one page break hooks
- **Async boundary**: entry imports `bootstrap.tsx` dynamically so
  shared modules negotiate before the app renders
- **Two shells** over one core: `shells/standalone` (own root) and
  `shells/embedded` (exposed component) — the app logic never knows
  which host it runs in
- **CSS isolation** for embedded remotes: prefix selectors under a
  theme root class (`postcss-prefix-selector`) so host and remote
  styles never collide
- MF 2.0 type hints (`@mf-types`) shared from remote to host — the
  contract between apps is typed

## HTTP & Realtime
- **fetch** + a thin typed client per backend (base URL, auth,
  schema parse of every response)
- **socket.io-client** or native `WebSocket` per backend protocol —
  shared as a federation singleton when used by remotes

## Dates & Utilities
- **Temporal** (where shipped) or **date-fns**; never Moment
- Platform first: `structuredClone`, `Array.prototype.toSorted`,
  `Object.groupBy`, `Intl.*` before a utility library

## Explicitly Avoided
- **Create React App, webpack for new apps** — unmaintained / slow;
  Vite 8 or Rsbuild 2
- **Decorator-based DI containers** in frontend code — runtime
  metadata, not erasable, heavy; pass dependencies as props/arguments
- **Moment, full lodash** — bundle weight the platform replaced
- **Redux for local UI state** — local state or Zustand
- **Runtime CSS-in-JS** — see Styling

## Selection Philosophy

### Priority Order
1. **Speed** — native implementation (Rust/Go/Zig) wins when it exists
2. **Bundle cost** — browser code pays for every kilobyte; check
   tree-shaken size before adopting
3. **Type inference** — the API must infer, not demand annotations
4. **Standards** — Standard Schema, Web APIs, ESM-only packages
5. **Active maintenance** — check last release before adopting

### Avoid
- Packages without ESM builds or types
- Libraries that need `any` at their call sites to compile
- Heavy dependency trees for trivial needs (`bun pm ls` / `pnpm why`)
