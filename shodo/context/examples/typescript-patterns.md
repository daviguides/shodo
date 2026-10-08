# TypeScript Patterns and Best Practices

## Guard Clauses (Flat over Nested)

Validate and return early — no arrow-shaped nesting:

```typescript
export function checkoutSummary(cart: Cart, user: User | undefined): Summary {
  if (user === undefined) {
    return { status: 'anonymous' };
  }
  if (cart.items.length === 0) {
    return { status: 'empty' };
  }

  // Main logic — flat and clear
  return { status: 'ready', total: cartTotal(cart.items, user.tier) };
}
```

## Parse at the Boundary, Trust Inside

The schema turns `unknown` into a typed value once. The type is
derived from the schema, never written twice:

```typescript
import * as v from 'valibot';

export const OrderSchema = v.object({
  id: v.pipe(v.string(), v.brand('OrderId')),
  amountCents: v.pipe(v.number(), v.integer(), v.minValue(0)),
  status: v.picklist(['pending', 'paid', 'shipped']),
});
export type Order = v.InferOutput<typeof OrderSchema>;

export async function fetchOrder(id: string, signal?: AbortSignal): Promise<Order> {
  const response = await fetch(`/api/orders/${id}`, { signal });
  if (!response.ok) {
    throw new HttpError(response.status, `GET /api/orders/${id}`);
  }
  return v.parse(OrderSchema, await response.json());
}
```

## Discriminated Union + Exhaustive Switch

```typescript
type TriageOutcome =
  | { kind: 'accepted'; taskId: TaskId }
  | { kind: 'deferred'; until: Date }
  | { kind: 'discarded'; reason: string };

// `satisfies never`: adding a variant fails compilation here
function describe(outcome: TriageOutcome): string {
  switch (outcome.kind) {
    case 'accepted': return `materialized as ${outcome.taskId}`;
    case 'deferred': return `parked until ${outcome.until.toISOString()}`;
    case 'discarded': return `dropped: ${outcome.reason}`;
    default: return outcome satisfies never;
  }
}
```

## Branded Identifiers

Swapped arguments become compile errors:

```typescript
type Brand<T, B extends string> = T & { readonly __brand: B };
export type UserId = Brand<string, 'UserId'>;
export type OrderId = Brand<string, 'OrderId'>;

declare function cancelOrder(user: UserId, order: OrderId): Promise<void>;

cancelOrder(orderId, userId); // ❌ compile error — arguments swapped
```

## Result Unions for Expected Failure

Exceptions for bugs; a result union when failure is a normal outcome
the caller must handle:

```typescript
type Result<T, E> = { ok: true; value: T } | { ok: false; error: E };

export function parseQuantity(raw: string): Result<number, 'not-a-number' | 'negative'> {
  const value = Number(raw);
  if (Number.isNaN(value)) return { ok: false, error: 'not-a-number' };
  if (value < 0) return { ok: false, error: 'negative' };
  return { ok: true, value };
}
```

## as const Instead of enum

```typescript
export const Status = {
  Pending: 'pending',
  Active: 'active',
  Done: 'done',
} as const;
export type Status = (typeof Status)[keyof typeof Status];

// satisfies keeps literal types while checking completeness
const statusLabel = {
  pending: 'Waiting',
  active: 'In progress',
  done: 'Completed',
} satisfies Record<Status, string>;
```

## Logic in Hooks, Rendering in Components

```typescript
// use-order-totals.ts — logic, testable without rendering markup
export function useOrderTotals(orderId: OrderId) {
  const { data: order, status } = useQuery({
    queryKey: ['order', orderId],
    queryFn: ({ signal }) => fetchOrder(orderId, signal),
  });
  const totals = order ? computeTotals(order) : undefined;
  return { totals, status };
}

// order-totals.tsx — rendering only
export function OrderTotals({ orderId }: OrderTotalsProps) {
  const { totals, status } = useOrderTotals(orderId);
  if (status === 'pending') return <Spinner />;
  if (totals === undefined) return <ErrorNotice />;
  return <TotalsTable totals={totals} />;
}
```

## Derive, Don't Sync

Values computable from props or state are computed during render —
never mirrored into state with an effect:

```typescript
function Cart({ items }: CartProps) {
  const [coupon, setCoupon] = useState<Coupon | undefined>();
  const total = cartTotal(items, coupon); // derived, no useEffect
  return <CartView items={items} total={total} onCoupon={setCoupon} />;
}
```

## Independent Work in Parallel

```typescript
const [user, orders, flags] = await Promise.all([
  fetchUser(userId, signal),
  fetchOrders(userId, signal),
  fetchFlags(signal),
]);
```

## Federated Remote: Async Boundary + Two Shells

```typescript
// src/index.ts — entry: only the async boundary
import('./bootstrap');

// src/bootstrap.tsx — standalone shell mounts its own root
import { createRoot } from 'react-dom/client';
import { StandaloneShell } from './shells/standalone/standalone-shell';

createRoot(document.getElementById('root')!).render(<StandaloneShell />);

// src/shells/embedded/embedded-app.tsx — exposed to the host
export default function EmbeddedApp(props: EmbeddedAppProps) {
  return (
    <div className="avo-theme">
      <AppCore config={props.config} />
    </div>
  );
}
```

`AppCore` (in `core/` + `ui/`) never knows which shell mounted it.
The `!` on `getElementById` is the one tolerated assertion: the HTML
template guarantees the node, and a missing root must crash loudly.

## Typed Environment Config

Parsed once at startup, imported everywhere:

```typescript
// core/config/env.ts
const EnvSchema = v.object({
  VITE_API_URL: v.pipe(v.string(), v.url()),
  VITE_SOCKET_URL: v.pipe(v.string(), v.url()),
});

export const env = v.parse(EnvSchema, import.meta.env);
```

## TSDoc Pattern

```typescript
/**
 * Computes the cart total in cents, applying the tier discount.
 *
 * @throws {PricingError} when an item has no price for the tier
 */
export function cartTotal(items: readonly CartItem[], tier: Tier): number { ... }
```
