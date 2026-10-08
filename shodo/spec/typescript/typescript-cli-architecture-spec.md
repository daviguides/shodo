# TypeScript CLI Architecture Specification

TypeScript materialization of the shodo CLI architecture, kept lean:
TypeScript is a frontend language in shodo, and CLIs here are the
exception (dev tooling, build scripts, small utilities). Same layers
and dependency rules as the Python spec.

**Existing CLIs keep their framework and runtime.** The choices below
apply to new CLIs.

## Runtime and Distribution

- **Node 24+ native type stripping** runs `.ts` directly from the
  repo (`node src/main.ts`) — no build step for internal tools
  (requires `erasableSyntaxOnly`). Node refuses to strip types inside
  `node_modules`, so a published package ships built JS
- **`bun build --compile`** for a single self-contained binary when
  the CLI is distributed — fastest startup, no runtime install
- Never require users to run `npx tsx …` for an installed tool

## Package Structure

```
project-name/
├── package.json             # "bin" → compiled binary or built JS
├── tsconfig.json
└── src/
    ├── main.ts              # Entrypoint — runMain(root) only
    ├── cli/                 # citty command definitions
    ├── commands/            # Interface layer (command runners)
    ├── services/            # Business logic layer
    ├── model/               # Types + schemas (data representation)
    ├── integrations/        # Low-level I/O (fs, subprocess, HTTP)
    ├── display/             # Semantic terminal output
    └── data/                # Bundled config
```

## Layered Architecture

```
cli/ → commands/ → services/ → model/
                             → integrations/
                             → data/
```

**`main.ts`** — `runMain(rootCommand)`. Zero logic. Under 20 lines.

**`cli/`** — `defineCommand` definitions: args, descriptions,
subcommands. Each `run` is one line: hand parsed args to a
`commands/` runner.

**`commands/`** — Interface layer. Turns args into a typed request,
calls services, formats via `display/`, maps domain errors to exit
codes.

**`services/`** — Business logic. **No citty imports. No
`console.log`.** Receives typed values, returns typed values or throws
typed errors.

**`model/`** — Types and Standard Schema definitions. No I/O.

**`integrations/`** — `node:fs/promises`, `node:child_process`,
`fetch`, YAML/JSON I/O. Domain-agnostic.

**`display/`** — `printSuccess`, `printError`, `printWarning` over
**picocolors**. Errors to stderr, results to stdout.

## Dependency Rules

| Layer | Can import from | Cannot import from |
|-------|----------------|-------------------|
| `cli/` | `commands/`, citty | `services/`, `integrations/` |
| `commands/` | `services/`, `model/`, `display/` | citty |
| `services/` | `model/`, `integrations/` | `cli/`, `commands/`, `display/`, citty |
| `model/` | schema library | anything else |
| `integrations/` | Node built-ins, third-party | `cli/`, `commands/`, `services/`, `model/` |
| `display/` | picocolors | `commands/`, `services/` |

## Entrypoint Convention

```typescript
// src/main.ts
import { runMain } from 'citty';
import { rootCommand } from './cli/root.ts';

await runMain(rootCommand);
```

```typescript
// src/cli/scan.ts — definitions only
import { defineCommand } from 'citty';
import { runScan } from '../commands/scan.ts';

export const scanCommand = defineCommand({
  meta: { name: 'scan', description: 'Scan repos for uncommitted changes.' },
  args: {
    all: { type: 'boolean', description: 'Include repos with no remote.' },
  },
  run: ({ args }) => runScan({ includeAll: args.all }),
});
```

- **citty** for new CLIs — tiny, zero-dependency, typed args;
  **commander** where the project already uses it
- Relative imports with explicit `.ts` extensions
  (`allowImportingTsExtensions` + `rewriteRelativeImportExtensions`) —
  Node's type stripping does not resolve tsconfig `paths` aliases

## Sub-Apps

When command groups develop distinct domains, graduate to vertical
slices (`src/commit/`, `src/sync/`) that replicate only the layers
they need, with shared domain in `src/core/`. Slices import from
`core/`, never from each other — same rules as the Python spec.

## package.json Convention

```json
{
  "name": "toolname",
  "type": "module",
  "bin": { "toolname": "./dist/toolname" },
  "scripts": {
    "dev": "node src/main.ts",
    "build": "bun build src/main.ts --compile --outfile dist/toolname"
  },
  "engines": { "node": ">=24" },
  "dependencies": { "citty": "^0.1", "picocolors": "^1" }
}
```
