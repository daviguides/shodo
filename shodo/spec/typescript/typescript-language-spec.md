# TypeScript Language Specification

TypeScript in shodo is a **frontend language**: browser applications,
React components, federated micro-frontends. Server and CLI use is
secondary (see typescript-cli-architecture-spec).

Types are **erased at runtime and unsound by design** — the type
system proves shape only inside the trust boundary. Everything that
enters from outside (HTTP, `JSON.parse`, storage, URL, env, message
events) is `unknown` until a schema parses it.

## TypeScript Version

### Required Version
- **TypeScript 7** (native compiler, stable since Jul 2026) for new
  projects — `tsc` is the Go port, ~10x faster type-checks
- **TypeScript 6 stays valid** where a tool still needs the JS
  compiler API (typescript-eslint, framework language servers, ts-morph)
  until TS 7.1 ships the stable API
- **Existing projects keep their TypeScript version** — upgrading the
  compiler is a deliberate project decision, never a side effect of a
  standards review
- Pin the version in `package.json` (exact or `~`), never `latest`

## Compiler Configuration

### tsconfig Baseline (New Projects)

```jsonc
{
  "compilerOptions": {
    "target": "ES2023",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "verbatimModuleSyntax": true,
    "erasableSyntaxOnly": true,
    "isolatedModules": true,
    "skipLibCheck": true,
    "noEmit": true,
    "paths": { "@/*": ["./src/*"] }
  }
}
```

- **`strict` is mandatory** in every project, new or existing
- `noUncheckedIndexedAccess` + `exactOptionalPropertyTypes` are the
  baseline for new projects — they close the holes kinhin's
  typescript-tdd-spec relies on the compiler to close
- **Existing projects keep their tsconfig** — tightening flags is a
  planned migration, not a review comment
- `verbatimModuleSyntax` + `erasableSyntaxOnly`: the source must be
  strippable by any native transformer (Oxc, SWC, Node type stripping)
  without type information

## Modern Language Features (Mandatory)

### Discriminated Unions for Variants
- **Discriminated unions** with a literal `kind`/`status` field for
  every closed set of variants — the TypeScript equivalent of sealed
  types
- **Exhaustive `switch`** with a `never` check — adding a variant
  becomes a compile error at every site

```typescript
type FetchState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

function describe(state: FetchState<User>): string {
  switch (state.status) {
    case 'idle': return 'waiting';
    case 'loading': return 'loading…';
    case 'success': return state.data.name;
    case 'error': return state.error.message;
    default: return state satisfies never;
  }
}
```

### No enum, No namespace
- **Never `enum`** or `namespace` — they emit runtime code and break
  `erasableSyntaxOnly`
- `as const` objects + derived union types replace enums

```typescript
export const Role = { Admin: 'admin', Member: 'member' } as const;
export type Role = (typeof Role)[keyof typeof Role];
```

### satisfies over Annotation for Literals
- `satisfies` checks a value against a type **without widening** it —
  config maps, route tables, theme tokens

### Branded Types for Domain Identifiers
- Brand IDs and validated primitives so swapped arguments fail to
  compile (`UserId` vs `OrderId`, `Email` vs `string`)
- The brand is applied only by the parser/constructor that validates

## Type Safety Discipline

### The Trust Boundary
- **`unknown` at every boundary**, narrowed by a schema parse — never
  annotate untrusted data with the type you hope it has
- **Never `any`** in production code. `unknown` + narrowing instead
- **`as` only at justified boundaries** (DOM APIs, third-party gaps),
  each with a comment stating why the cast is sound — every cast is a
  hole kinhin requires tests to cover
- **No non-null assertion `!`** — narrow with a guard or restructure
- Runtime validation methodology and contract tests:
  `@~/.claude/kinhin/spec/typescript-tdd/typescript-tdd-spec.md`

### Inference vs Annotation
- **Annotate exported signatures** (parameters and return types) —
  they are the module's API
- Let inference handle locals and private helpers
- `readonly` on props, parameters, and data that must not mutate

## Null Handling

- **`undefined` for absence** in application code; `null` only where
  an external API or the DOM returns it
- Optional chaining (`?.`) and nullish coalescing (`??`) — never `||`
  for defaults (it swallows `0` and `''`)
- Under `exactOptionalPropertyTypes`, `prop?: T` means "may be
  missing", not "may be `undefined`" — model both explicitly when both
  occur

## Error Handling

### Errors Are Typed at the Edges
- **Throw `Error` subclasses** with a fixed `name` and `{ cause }`
  chaining — never throw strings or plain objects
- **Result unions at module boundaries** where failure is an expected
  outcome (parsing, network) — exceptions stay for bugs and
  unrecoverable states
- `catch (error: unknown)` — narrow before reading anything
- Never empty `catch` — handle, rethrow with cause, or report and
  explain why suppression is safe

```typescript
export class ConfigError extends Error {
  override name = 'ConfigError';
  constructor(message: string, options?: ErrorOptions) {
    super(message, options);
  }
}

throw new ConfigError(`cannot parse config at ${url}`, { cause: error });
```

### Promises
- **Every promise is awaited, returned, or explicitly voided** —
  floating promises are defects
- `Promise.all` / `Promise.allSettled` for independent work —
  never sequential `await` in a loop over independent items
- `AbortController` for cancellable requests and effects

## Modules and Imports

- **ESM only** — no CommonJS in new code
- `import type` / `export type` for type-only symbols (enforced by
  `verbatimModuleSyntax`)
- Path alias `@/` for cross-feature imports; relative imports inside
  a feature folder
- **No barrel files** (`index.ts` re-exporting a folder) inside the
  app — they defeat tree-shaking and slow dev-server startup; a
  package's public entry is the exception
- Named exports by default; default export only where a tool demands
  it (lazy route modules, federation `exposes`)

## Documentation

### TSDoc on Exported API
- TSDoc (`/** */`) on exported functions, hooks, components, and
  types — compact, English, first sentence is the summary
- Do not restate types already in the signature
- `@throws` when a function throws by design

## Constants

### No Magic Values
- `UPPER_SNAKE_CASE` for module-level primitive constants;
  `as const` objects for related groups
- Environment values via a single validated config module
  (`import.meta.env` parsed once at startup), never read ad hoc

## Integration Philosophy

### Modern TypeScript Principles
- **Make invalid states unrepresentable** — discriminated unions and
  brands over boolean flags and optional soup
- **Parse, don't validate** — the boundary turns `unknown` into a
  typed value once; the interior trusts it
- **Erasable TypeScript** — nothing in the source needs the type
  checker to run; native toolchains strip it
- **Immutability by default** — `readonly`, spread updates, no
  mutation of props or state
