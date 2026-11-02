# TypeScript Make Targets

Contract: `spec/make-contract-spec.md`. Use the package manager the
component's lockfile names (`pnpm-lock.yaml` → pnpm, `package-lock.json`
→ npm, `yarn.lock` → yarn, `bun.lockb` → bun). Never run an install in
`test` or `check`.

| Target | Commands |
|--------|----------|
| `test` | the package's test script, e.g. `pnpm test` / `npm test` (vitest, jest) in non-watch mode |
| `check` | `<pm> exec tsc --noEmit`; the lint script when configured (`<pm> run lint`, ESLint/Biome); the format check when configured (`prettier --check` / `biome check`) |

```make
.PHONY: install test check format

install:
	pnpm install --frozen-lockfile

test:
	pnpm test --run

check:
	pnpm exec tsc --noEmit
	pnpm run lint

format:
	pnpm run format
```

The test runner MUST NOT start in watch mode under `make test`
(`vitest --run`, `jest --ci`). A component without a lint config omits
the lint line; it never adds `|| true`.
