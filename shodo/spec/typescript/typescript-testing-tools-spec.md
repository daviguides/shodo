# TypeScript Testing Tools Specification

Tooling only. TDD methodology (trust boundary, RED includes type
errors, module mocks as scaffold smell, lifecycle tags, runner
contract) lives in kinhin:
`@~/.claude/kinhin/spec/typescript-tdd/typescript-tdd-spec.md`

**Existing projects keep their test runner.** A Jest project stays on
Jest; the tools below are the choice for new projects.

## Test Runner

### Vitest 4
- **Vitest 4** — reuses the Vite/Rolldown transform pipeline, so
  tests compile like the app; native-speed transforms via Oxc
- Rsbuild projects use Vitest too (or Rstest when the project already
  adopted it) — the runner choice is independent of the bundler
- Tests next to the code: `cart-total.ts` + `cart-total.test.ts`
- Names state behavior (`applies loyalty discount for premium users`)

```typescript
import { describe, expect, it } from 'vitest';

describe('cartTotal', () => {
  it('applies the premium discount', () => {
    const total = cartTotal([item(100)], { tier: 'premium' });

    expect(total).toBe(90);
  });
});
```

### Type-Checking Is a Separate Step
- Vitest strips types without checking them — **`tsc --noEmit` runs
  separately** (kinhin Principle 2)
- `expectTypeOf` (built into Vitest) for exported type contracts

## Component Tests

### Testing Library
- **@testing-library/react** + **@testing-library/user-event** —
  query by role and accessible name, interact like a user
- Never query by class name or test id when a role exists
- Environment: **happy-dom** (faster) or jsdom — follow the project

```typescript
it('submits the order', async () => {
  const user = userEvent.setup();
  const onSubmit = vi.fn();
  render(<OrderForm onSubmit={onSubmit} />);

  await user.type(screen.getByRole('spinbutton', { name: /quantity/i }), '2');
  await user.click(screen.getByRole('button', { name: /place order/i }));

  expect(onSubmit).toHaveBeenCalledWith({ quantity: 2 });
});
```

### Browser Mode
- **Vitest Browser Mode** (stable in Vitest 4, Playwright provider)
  when a component depends on real layout, focus, or browser APIs that
  happy-dom fakes poorly
- Visual regression via browser-mode screenshots for design-system
  components only

## Network — MSW
- **Mock Service Worker** at the network boundary — the same handlers
  serve tests and local development
- Never mock `fetch` by hand per test; never mock the app's own API
  client module (kinhin: inject or intercept at the boundary)

## End-to-End — Playwright
- **Playwright** for critical user journeys only (login, checkout,
  federation host ↔ remote handshake)
- Run against a production build (`vite preview` / `rsbuild preview`)
- Federated apps: one e2e per exposed module mounted in a real host

## Mutation Testing — Stryker
- **StrykerJS** with the Vitest runner, scoped to changed files
  (kinhin runner contract)

## Coverage
- **`@vitest/coverage-v8`** — native V8 coverage, no instrumentation
  pass

**Targets** (same bar as the Python spec):
- Overall: ≥ 90%
- Business logic (pure functions, hooks with logic): 100%
- Presentational components: covered by behavior tests, not by line
  count

## Configuration Pattern

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'happy-dom',
    setupFiles: ['./src/test/setup.ts'],
    coverage: { provider: 'v8', thresholds: { lines: 90 } },
  },
});
```

Runner flags (bail, shuffle, reporters, pool) follow kinhin's runner
configuration — not repeated here.

## CI Commands

```bash
<fmt> --check            # oxfmt / biome / prettier — whatever the project uses
<lint>                   # oxlint --type-aware / eslint
tsc --noEmit
vitest run --coverage
playwright test          # e2e suite
```

## Best Practices
- Arrange / act / assert separated by blank lines
- Factories for test data (`item(100)`), not shared mutable fixtures
- Test the hook through a component or `renderHook` — never its
  internal state
- No snapshot of rendered markup as a permanent test (kinhin)
