# TypeScript Anti-Patterns and Corrections

## Anti-Pattern 1: Typing Untrusted Data with a Hope

### ❌ ANTI-PATTERN
```typescript
const order: Order = await response.json();   // compiles, proves nothing
const user = JSON.parse(raw) as User;          // cast reopens the hole
```

### ✅ CORRECT
```typescript
const order = v.parse(OrderSchema, await response.json());
```

**Why this matters:**
- `response.json()` returns `any`; the annotation is a claim, not a check
- A malformed payload travels deep into the app before it explodes
- The schema is the one place where `unknown` becomes typed

## Anti-Pattern 2: any and Casual as

### ❌ ANTI-PATTERN
```typescript
function handle(event: any) {
  const value = (event.target as any).value as number;
}
```

### ✅ CORRECT
```typescript
function handleChange(event: React.ChangeEvent<HTMLInputElement>) {
  const value = event.currentTarget.valueAsNumber;
}
```

**Why this matters:**
- `any` switches the type checker off for everything it touches
- Each `as` is a test kinhin now requires, because the compiler stopped
  proving the shape
- The typed event API already carries the right type

## Anti-Pattern 3: enum

### ❌ ANTI-PATTERN
```typescript
enum Status { Pending, Active }   // emits runtime code, numeric values
```

### ✅ CORRECT
```typescript
const Status = { Pending: 'pending', Active: 'active' } as const;
type Status = (typeof Status)[keyof typeof Status];
```

**Why this matters:**
- `enum` is not erasable — it breaks native type stripping
  (`erasableSyntaxOnly`)
- Numeric enums accept any number; string unions accept only the
  listed values

## Anti-Pattern 4: Boolean Soup Instead of a Union

### ❌ ANTI-PATTERN
```typescript
type State = {
  isLoading: boolean;
  isError: boolean;
  data?: User;
  error?: Error;
};   // isLoading && isError && data — representable, meaningless
```

### ✅ CORRECT
```typescript
type State =
  | { status: 'loading' }
  | { status: 'error'; error: Error }
  | { status: 'success'; data: User };
```

**Why this matters:**
- Four fields allow sixteen combinations; three are valid
- The union makes the invalid thirteen unrepresentable

## Anti-Pattern 5: Floating Promises

### ❌ ANTI-PATTERN
```typescript
function onSave() {
  saveDraft(draft);   // rejection becomes an unhandled error
  close();
}
```

### ✅ CORRECT
```typescript
async function onSave() {
  await saveDraft(draft);
  close();
}
```

**Why this matters:**
- The dialog closes before the save completes or fails
- The type-aware lint rule `no-floating-promises` catches this; keep it on

## Anti-Pattern 6: useEffect to Sync Derived State

### ❌ ANTI-PATTERN
```typescript
const [total, setTotal] = useState(0);
useEffect(() => {
  setTotal(cartTotal(items));
}, [items]);   // extra render, one frame of stale total
```

### ✅ CORRECT
```typescript
const total = cartTotal(items);
```

**Why this matters:**
- Derived values belong to render, not to state
- Effects are for synchronizing with external systems only

## Anti-Pattern 7: Hand-Rolled Fetch in useEffect

### ❌ ANTI-PATTERN
```typescript
useEffect(() => {
  fetch(`/api/orders/${id}`).then((r) => r.json()).then(setOrder);
}, [id]);   // no abort, race on fast id changes, no cache, no error
```

### ✅ CORRECT
```typescript
const { data: order } = useQuery({
  queryKey: ['order', id],
  queryFn: ({ signal }) => fetchOrder(id, signal),
});
```

**Why this matters:**
- Out-of-order responses overwrite fresh data with stale data
- TanStack Query adds abort, dedup, cache, and retry for free

## Anti-Pattern 8: Swallowed Errors

### ❌ ANTI-PATTERN
```typescript
try {
  return parseConfig(raw);
} catch {
  return {};   // broken config silently becomes empty config
}
```

### ✅ CORRECT
```typescript
try {
  return parseConfig(raw);
} catch (error) {
  throw new ConfigError('invalid config', { cause: error });
}
```

## Anti-Pattern 9: Barrel Files Inside the App

### ❌ ANTI-PATTERN
```typescript
// features/checkout/index.ts
export * from './cart';
export * from './payment';
export * from './shipping';   // importing one symbol loads all three
```

### ✅ CORRECT
```typescript
import { cartTotal } from '@/features/checkout/cart-total';
```

**Why this matters:**
- The dev server transforms every re-exported module on first import
- Circular imports hide behind barrels

## Anti-Pattern 10: Duplicated Singletons in a Federated Remote

### ❌ ANTI-PATTERN
```typescript
pluginModuleFederation({
  name: 'assistant',
  exposes: { './EmbeddedApp': './src/shells/embedded/embedded-app.tsx' },
  // react not shared — the host and the remote each load their own
});
```

### ✅ CORRECT
```typescript
shared: {
  react: { singleton: true, eager: true },
  'react-dom': { singleton: true, eager: true },
},
```

**Why this matters:**
- Two Reacts on one page break hooks ("Invalid hook call")
- Context from the host never reaches the remote's tree

## Anti-Pattern 11: Rewriting an Existing Project's Toolchain

### ❌ ANTI-PATTERN
```
Review comment: "Replace ESLint with oxlint and npm with bun — faster."
```

### ✅ CORRECT
```
Follow the project's ESLint + npm setup as defined.
```

**Why this matters:**
- The toolchain of an existing project is a decision with context
  (federation needs, CI, team, compatibility) a review does not see
- "Fastest" guides new projects; it never overrides an established one
