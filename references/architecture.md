# Architecture

## Dependency direction

A common top-down structure is:

```text
Pages
  ↓
Components / Hooks
  ↓
APIs
  ↓
Utils / Constants
  ↓
Types
```

The exact folder names may differ by project, but dependency direction should remain clear.

- Higher layers may depend on lower layers.
- Lower layers must not import higher layers.
- Avoid circular dependencies.

## Responsibilities

### Pages
Compose a page or feature and coordinate high-level behavior.

### Components
Render UI and coordinate component-level interaction.

### Hooks
Encapsulate reusable React behavior and state access.

### APIs
Own communication with backend or external services.

### Utils / constants
Contain genuinely reusable lower-level helpers and stable domain constants.

### Types
Contain shared TypeScript contracts when they need a lower-level shared home.

## Before introducing architecture

Discover the existing architecture first:

1. Find nearby implementations solving a similar problem.
2. Identify ownership of state and business rules.
3. Reuse established boundaries when they fit.
4. Change an existing authoritative source instead of creating a second source.
5. Introduce a new abstraction only when its semantic responsibility is clear.
6. Avoid unrelated refactors.

Architecture should make dependency direction and ownership easy to understand.

## Examples

### Dependency direction: lower layers never import higher ones

```ts
// BAD: a util imports from a component (lower layer -> higher layer)
// utils/formatPrice.ts
import { CurrencyContext } from '../components/CurrencyProvider';

// GOOD: pass what the util needs as an argument
// utils/formatPrice.ts
export function formatPrice(amount: number, currency: string) {
  return new Intl.NumberFormat(undefined, { style: 'currency', currency }).format(amount);
}
```

```ts
// BAD: circular dependency
// api/orders.ts      -> imports  hooks/useOrderFilters.ts
// hooks/useOrderFilters.ts -> imports  api/orders.ts

// GOOD: shared contract moves down to a lower layer both can import
// types/order.ts       <- imported by api/orders.ts and hooks/useOrderFilters.ts
```

### Responsibilities: keep each layer to its job

```tsx
// BAD: component owns backend communication details
function OrderList() {
  const [orders, setOrders] = useState<Order[]>([]);
  useEffect(() => {
    fetch('/api/orders', { headers: { Authorization: `Bearer ${token}` } })
      .then((r) => r.json())
      .then(setOrders);
  }, []);
  return <Table rows={orders} />;
}

// GOOD: API layer owns the request, hook exposes it, component renders
// api/orders.ts
export async function getOrders(): Promise<Order[]> {
  const res = await http.get('/orders');
  return res.data;
}

// hooks/useOrders.ts
export const useOrders = () => useQuery({ queryKey: ['orders'], queryFn: getOrders });

// components/OrderList.tsx
function OrderList() {
  const { data = [] } = useOrders();
  return <Table rows={data} />;
}
```

### Discover before introducing

```text
BAD:  Task: "add a Coupons admin list."
      Create a new `useTableState` hook and a custom pagination component
      without looking at how Orders and Users lists already do it.

GOOD: Find the closest existing list (Orders). Reuse its table, filter, and
      pagination pattern. Add only what Coupons genuinely needs on top.
```

### Change the existing source instead of creating a second one

```ts
// BAD: new constants file duplicating an existing one
// constants/orderStatusLabels.ts   (new)
// constants/orderStatus.ts         (already has the labels)

// GOOD: extend the authoritative definition
// constants/orderStatus.ts
export const orderStatusMeta = {
  pending: { title: 'Pending' },
  paid: { title: 'Paid' },
  refunded: { title: 'Refunded' }, // added here
};
```
