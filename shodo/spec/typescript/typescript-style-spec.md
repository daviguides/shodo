# TypeScript Style Specification

## Toolchain Philosophy

### Flexible, Native-First
The JavaScript toolchain has many competing currents. Unlike the
Python, Rust, and Java specs, this spec does **not** prescribe one
tool per category. It prescribes a rule for choosing:

1. **Existing project → its toolchain is law.** Package manager,
   bundler, linter, formatter, and test runner already defined in the
   project are NEVER changed by a standards pass or a review. Follow
   them as they are. Suggesting a migration is out of scope unless the
   user asks for one.
2. **New project, or a category the project lacks → the fastest
   option.** In practice that means **native tooling** (Rust, Go, Zig)
   — the same reason the Python stack is uv, ruff, pydantic-core, and
   polars. Recommend it; do not impose it on an existing project that
   simply lacks the category.

### Fastest Option per Category (state: Oct 2026)

| Category | Fastest (new projects) | Native | Maturity note |
|----------|------------------------|--------|---------------|
| Package manager | **bun install** | Zig | Fastest raw installs; `bun.lock` not portable |
| Package manager (alt) | **pnpm 12** | Rust | Stable Aug 2026; mature workspaces, strict isolation |
| Bundler | **Vite 8** (Rolldown) | Rust | Stable; default for non-federated apps |
| Bundler (federation) | **Rsbuild 2** (Rspack 2) | Rust | Best Module Federation 2.0 support |
| Linter | **oxlint** (+ `oxlint-tsgolint` for type-aware) | Rust/Go | Stable; type-aware rules via TS native |
| Formatter | **oxfmt** | Rust | Beta, 100% Prettier JS/TS conformance |
| Formatter (stable) | **Biome** | Rust | Stable; choose when beta is unacceptable |
| Type-check | **tsc** from TypeScript 7 | Go | Stable; typescript-eslint not compatible yet |
| Test runner | **Vitest 4** | Rolldown/Oxc pipeline | Reuses the Vite transform; browser mode stable |

The table is a dated snapshot. Re-verify before recommending: native
tools in this space ship majors every few months.

## Formatting

- **The formatter owns layout** — never hand-format, never argue
  style in review
- Whatever formatter the project has (oxfmt, Biome, Prettier) runs
  locally and in CI as a check
- Defaults over configuration: 2-space indent, single quotes,
  trailing commas, 80–100 columns — accept the tool's defaults unless
  the project already overrides them
- A project with no formatter: recommend oxfmt (or Biome), but do not
  reformat files outside the change being made

## Linting

### Lint as Law
- Lint runs in CI and fails the build — rules are not suggestions
- Type-aware rules that matter: no floating promises, no misused
  promises, no unnecessary conditions, exhaustive switch,
  `consistent-type-imports`
- Suppress a rule only at the smallest scope, with a comment stating
  why (`// oxlint-disable-next-line no-explicit-any -- third-party gap`)
- **typescript-eslint does not support TypeScript 7** yet — an ESLint
  project stays on TS 6 for linting, or uses oxlint's type-aware mode

## Naming Conventions

| Item | Convention |
|------|------------|
| Variables, functions, hooks | `camelCase` |
| Types, interfaces, components, classes | `PascalCase` |
| Module-level constants | `UPPER_SNAKE_CASE` |
| `as const` lookup objects | `PascalCase` (used like a type) |
| Files and folders | `kebab-case.ts`; component files follow the project convention |
| Type parameters | `T`, `K`, `V` or descriptive `TItem` |

### Frontend-Specific Naming
- Hooks start with `use` — `useOrderTotals`, never `getOrderTotals`
  for something that calls hooks
- Event props `onX` (`onSubmit`), handlers `handleX` (`handleSubmit`)
- Booleans in question form: `isOpen`, `hasError`, `canRetry`
- **No `I` prefix** on interfaces, no `T` prefix on types
- `type` for unions, mapped types, and props; `interface` when
  declaration merging or `extends` chains are wanted — follow the
  project's existing choice

## File Organization

### Feature Folders
- Group by **feature**, not by technical kind — `features/checkout/`
  holds its components, hooks, services, and types together
- Shared building blocks in `ui/` (presentational) and `core/`
  (config, services, types) — the avo-assistant-ui layout
- Tests live next to the code: `order-total.ts` +
  `order-total.test.ts`

### Size Limits
- One exported component per file
- File > ~300 lines or function/component > ~50 lines: split by
  responsibility
- More than ~3 positional parameters: take one options object

## Component Style

- **Function components only** — no class components
- Props typed with a `type XProps`, destructured in the signature
- Logic in custom hooks, rendering in components — a component that
  fetches, transforms, and renders is three responsibilities
- No inline object/array literals as props in hot paths when the
  project does not use the React Compiler

## Comments

- **English only**, explain **why**, not what
- `// TODO(context):` / `// FIXME(context):` for tracked debt
- TSDoc for exported API; `//` for internal reasoning
- Never leave commented-out code

## Consistency Enforcement

- Format check, lint, and `tsc --noEmit` in CI — all three, every PR
- Whatever runner the project uses for these stays; the CI shape
  (format → lint → type-check → test) is the standard
