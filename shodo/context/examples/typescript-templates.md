# TypeScript Code Templates

## Service Function Template

```typescript
/**
 * {Brief description of what this function does}.
 *
 * @throws {{DomainError}} when {condition}
 */
export async function {functionName}(
  {param1}: {Type1},
  options: {FunctionName}Options = {},
): Promise<{ReturnType}> {
  // Guard clauses first (flat > nested)
  if (!{precondition}) {
    throw new {DomainError}(`{context}: ${{param1}}`);
  }

  // Main logic
  const {intermediate} = await {stepOne}({param1}, options.signal);
  return {stepTwo}({intermediate});
}
```

## Schema + Derived Type Template

```typescript
import * as v from 'valibot';

/** {What this value represents}. */
export const {Name}Schema = v.object({
  id: v.pipe(v.string(), v.brand('{Name}Id')),
  {field}: v.pipe(v.string(), v.minLength(1)),
  {status}: v.picklist([{'a'}, {'b'}]),
});

export type {Name} = v.InferOutput<typeof {Name}Schema>;
```

## Discriminated Union Template

```typescript
/** Outcome of {operation}. */
export type {Operation}Result =
  | { kind: 'ok'; {payload}: {Type} }
  | { kind: 'failed'; reason: string; cause?: unknown };
```

## Domain Error Template

```typescript
export class {Domain}Error extends Error {
  override name = '{Domain}Error';

  constructor(message: string, options?: ErrorOptions) {
    super(message, options);
  }
}
```

## Component Template

```typescript
type {Component}Props = {
  readonly {prop}: {Type};
  readonly on{Event}?: ({arg}: {ArgType}) => void;
};

/** {What this component renders}. */
export function {Component}({ {prop}, on{Event} }: {Component}Props) {
  const { {value}, status } = use{Logic}({prop});

  if (status === 'pending') return <{Skeleton} />;

  return (
    <section aria-label="{accessible name}">
      {/* rendering only — logic lives in use{Logic} */}
    </section>
  );
}
```

## Custom Hook Template

```typescript
/** {What state/behavior this hook provides}. */
export function use{Logic}({input}: {InputType}) {
  const query = useQuery({
    queryKey: ['{resource}', {input}],
    queryFn: ({ signal }) => fetch{Resource}({input}, signal),
  });

  const {derived} = query.data ? compute{Derived}(query.data) : undefined;
  return { {derived}, status: query.status };
}
```

## Test Template

```typescript
import { describe, expect, it } from 'vitest';

describe('{unitUnderTest}', () => {
  it('{states the behavior}', () => {
    const {input} = {factory}({overrides});

    const result = {unitUnderTest}({input});

    expect(result).toEqual({expected});
  });
});
```

## tsconfig.json Template (New Project)

```jsonc
{
  "compilerOptions": {
    "target": "ES2023",
    "lib": ["ES2023", "DOM", "DOM.Iterable"],
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
  },
  "include": ["src"]
}
```

## vite.config.ts Template (Non-Federated App)

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: { alias: { '@': new URL('./src', import.meta.url).pathname } },
});
```

## rsbuild.config.ts Template (Federated Remote)

```typescript
import { defineConfig } from '@rsbuild/core';
import { pluginReact } from '@rsbuild/plugin-react';
import { pluginModuleFederation } from '@module-federation/rsbuild-plugin';

const isRemote = process.env.RSBUILD_MF_REMOTE === 'true';

export default defineConfig({
  plugins: [
    pluginReact(),
    ...(isRemote
      ? [
          pluginModuleFederation({
            name: '{remote_name}',
            exposes: {
              './EmbeddedApp': './src/shells/embedded/embedded-app.tsx',
              './config': './src/core/config/index.ts',
            },
            shared: {
              react: { singleton: true, eager: true },
              'react-dom': { singleton: true, eager: true },
              '{design-system}': { singleton: true, eager: true },
            },
          }),
        ]
      : []),
  ],
  source: { entry: { index: './src/index.ts' } },
});
```

## package.json Scripts Template

```json
{
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc --noEmit && vite build",
    "typecheck": "tsc --noEmit",
    "lint": "oxlint --type-aware",
    "fmt": "oxfmt",
    "fmt:check": "oxfmt --check",
    "test": "vitest",
    "test:ci": "vitest run --coverage"
  }
}
```

Tool names in scripts are the new-project defaults; an existing
project keeps its own (`eslint`, `biome`, `prettier`, `jest`).
