# Reusability

## Generic-first extraction

When implementing a feature, look for logic that expresses a **generic concept** rather than a business-specific one, and foresee whether other features are likely to need the same logic.

If so, put it in a shared place (for example `@/utils`, `@/hooks`, `@/components`) with a **generic name** that does not mention the current business entity (no `Order`, `Product`, `Item`, `User` in the name), so a future feature can reuse it.

Apply this when:
- the logic is a real generic concept (formatting, parsing, grouping, pagination, confirmation, debouncing);
- reuse is plausible from the product domain, not merely imaginable;
- the generic version is not noticeably more complicated than the specific one.

Do not apply it when:
- the generic version needs many options, flags, or branches to cover hypothetical cases;
- the logic encodes a rule that is truly specific to one entity;
- the extraction would be a speculative framework for a single use case.

A good test: *"If `Order` or `ReturnRefund` needed this next month, would they import it as is?"*

## Look up before creating

Before writing any helper, hook, or component that looks generic, check whether one already exists:

1. Look in the shared folders (`@/utils`, `@/hooks`, `@/components`) and at files with relevant names (`currency.ts`, `price.ts`, `date.ts`).
2. Search for the behavior, not only the name (for example search `Intl.NumberFormat` or `toLocaleString`).
3. Decide:
   - **Exists in a shared place with a generic name** -> reuse it.
   - **Exists but is business-specific in name or location** -> rename it to a generic name, move it to a shared place, update its callers, then reuse it.
   - **Does not exist** -> create it generically in a shared place.
4. Keep the refactor limited to the helper and its direct callers.

## Don't hide different logic behind one reusable function

Reuse means the callers share the same logic. If a shared function's body branches on a parameter value (`if`/`else`, `switch`, a `type` or `mode` flag) to run different logic for each caller, it is several functions in one. Split it into separate, well-named functions.

Warning signs:
- a parameter such as `type`, `kind`, `mode`, or `variant` that selects different logic;
- boolean flags that switch behavior (`format(x, true)`);
- optional params that only matter for some values of another param;
- a body that is mostly one `if`/`switch` over the param, with little shared code between the branches.

How to fix it:
1. Give each branch its own function with a specific name.
2. If the branches share a real step, extract just that step as a small helper and let the separate functions call it.
3. Callers pick the function, instead of passing a flag for the function to interpret.

Branching is fine in two cases:
- **Data**: a lookup map, or a genuinely shared algorithm that takes data as input.
- **Dispatch**: a thin `switch`/`if` on a discriminant (such as an enum) where each branch only delegates to its own separate function. The branching picks a function. It doesn't hold the logic.

The problem is a function whose branches each contain different inline logic.

This complements generic-first extraction. Shared code is generic when callers can pass different data to the same logic. It is not generic when callers pass a flag to pick different logic.

## Examples

### Generic name, shared place

```tsx
// BAD: business-specific formatter hidden inside one component
// components/ProductPriceDisplay.tsx
function ProductPriceDisplay({ product }: { product: Product }) {
  const text = new Intl.NumberFormat(undefined, {
    style: 'currency',
    currency: product.currency,
  }).format(product.prod_price);
  return <span>{text}</span>;
}
// Later, OrderPrice and ReturnPrice copy the same Intl code.

// GOOD: generic formatter, no entity names, in a shared place
// utils/currency.ts
export function formatPriceWithCurrency(amount: number, currency: string) {
  return new Intl.NumberFormat(undefined, { style: 'currency', currency }).format(amount);
}

// components/ProductPriceDisplay.tsx
function ProductPriceDisplay({ product }: { product: Product }) {
  return <span>{formatPriceWithCurrency(product.prod_price, product.currency)}</span>;
}
// OrderPrice and ReturnPrice can import the same function.
```

### Generic hook

```ts
// BAD: debounce tied to one feature
function useProductSearchDebounce(keyword: string) {
  const [value, setValue] = useState(keyword);
  useEffect(() => {
    const id = setTimeout(() => setValue(keyword), 300);
    return () => clearTimeout(id);
  }, [keyword]);
  return value;
}

// GOOD: generic, reusable by any search or filter
// hooks/useDebouncedValue.ts
export function useDebouncedValue<T>(value: T, delayMs = 300) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delayMs);
    return () => clearTimeout(id);
  }, [value, delayMs]);
  return debounced;
}
```

### Generic pagination logic

```ts
// BAD
function getOrderPageCount(totalOrders: number, pageSize: number) {
  return Math.ceil(totalOrders / pageSize);
}

// GOOD
// utils/pagination.ts
export function getPageCount(totalItems: number, pageSize: number) {
  return Math.max(1, Math.ceil(totalItems / pageSize));
}
```

### Generic file download

```ts
// BAD: only invoices can use it
function downloadInvoicePdf(blob: Blob, invoiceNo: string) {
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `invoice-${invoiceNo}.pdf`;
  a.click();
  URL.revokeObjectURL(url);
}

// GOOD: the filename is a parameter; the domain naming stays with the caller
// utils/download.ts
export function downloadBlob(blob: Blob, filename: string) {
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  a.click();
  URL.revokeObjectURL(url);
}

downloadBlob(blob, `invoice-${invoiceNo}.pdf`);
```

### Generic UI component

```tsx
// BAD: delete confirmation hard-wired to orders
function DeleteOrderDialog({ order, onConfirm }: { order: Order; onConfirm: () => void }) {
  return (
    <Dialog title={`Delete order #${order.code}?`}>
      <button onClick={onConfirm}>Delete</button>
    </Dialog>
  );
}

// GOOD: generic dialog, the caller supplies the wording
type ConfirmDialogProps = {
  title: string;
  description?: string;
  confirmLabel?: string;
  onConfirm: () => void;
};
function ConfirmDialog({ title, description, confirmLabel = 'Confirm', onConfirm }: ConfirmDialogProps) {
  return (
    <Dialog title={title} description={description}>
      <button onClick={onConfirm}>{confirmLabel}</button>
    </Dialog>
  );
}

<ConfirmDialog title={`Delete order #${order.code}?`} confirmLabel="Delete" onConfirm={onDelete} />
```

### Generic collection helper

```ts
// BAD
const groupOrdersByStatus = (orders: Order[]) =>
  orders.reduce<Record<string, Order[]>>((acc, o) => {
    (acc[o.status] ??= []).push(o);
    return acc;
  }, {});

// GOOD
// utils/collection.ts
export function groupBy<T, K extends PropertyKey>(items: T[], getKey: (item: T) => K) {
  return items.reduce<Record<K, T[]>>((acc, item) => {
    (acc[getKey(item)] ??= []).push(item);
    return acc;
  }, {} as Record<K, T[]>);
}

const ordersByStatus = groupBy(orders, (o) => o.status);
```

### When NOT to generalize

```ts
// BAD: speculative, option-heavy "generic" function for one use
function formatValue(
  value: number,
  opts: { kind: 'price' | 'percent' | 'weight'; currency?: string; unit?: string; round?: boolean },
) {
  /* many branches for cases nobody needs yet */
}

// GOOD: keep it specific until a second real caller appears
const formatDiscountPercent = (rate: number) => `${Math.round(rate * 100)}%`;
```

```ts
// GOOD: truly entity-specific rule stays with its entity
const canRefundOrder = (order: Order) => order.status === 'paid' && !order.refundedAt;
```

### Look up before creating

```text
Task: "Show the price on the return-refund page."

BAD:
  Write a new `formatReturnPrice` in ReturnPrice.tsx without checking the repo.

GOOD, case A, a generic helper already exists in a shared place:
  `@/utils/currency.ts` exports `formatPriceWithCurrency`.
  -> Import and use it.

GOOD, case B, a helper exists but is specific or misplaced:
  `@/features/orders/formatOrderPrice.ts` does the same Intl formatting.
  -> Rename to `formatPriceWithCurrency`, move to `@/utils/currency.ts`,
     update its existing callers, then use it in ReturnPrice.
     Keep the change limited to this helper and its direct callers.

GOOD, case C, nothing exists:
  -> Create `formatPriceWithCurrency` in `@/utils/currency.ts` with a generic name.
```

### Split flag-driven functions

```ts
// BAD: one "reusable" function that is really three
function formatValue(value: number, type: 'price' | 'percent' | 'weight', currency?: string) {
  if (type === 'price') {
    return new Intl.NumberFormat(undefined, { style: 'currency', currency }).format(value);
  } else if (type === 'percent') {
    return `${Math.round(value * 100)}%`;
  } else {
    return `${value} kg`;
  }
}
formatValue(9.5, 'price', 'USD');

// GOOD: separate functions, each with a clear name and signature
formatPriceWithCurrency(9.5, 'USD');
formatPercent(0.25);
formatWeightInKg(2);
```

### Boolean flag that switches behavior

```ts
// BAD: caller has to know what `true` means; two behaviors in one body
function exportData(rows: Row[], asCsv: boolean) {
  if (asCsv) {
    return rows.map((r) => Object.values(r).join(',')).join('\n');
  }
  return JSON.stringify(rows);
}
exportData(rows, true);

// GOOD: separate functions
function toCsv(rows: Row[]) {
  return rows.map((r) => Object.values(r).join(',')).join('\n');
}
function toJson(rows: Row[]) {
  return JSON.stringify(rows);
}
```

### Component with a variant prop that branches the whole body

```tsx
// BAD: `type` decides which UI renders; each branch has its own props
function EntityCard({ type, data }: { type: 'product' | 'order' | 'user'; data: any }) {
  if (type === 'product') return <div>{data.name} - {formatPriceWithCurrency(data.price, data.currency)}</div>;
  if (type === 'order') return <div>#{data.code} - {data.status}</div>;
  return <div>{data.fullName} ({data.email})</div>;
}

// GOOD: separate components; share only the real common shell
function Card({ children }: { children: React.ReactNode }) {
  return <div className="card">{children}</div>;
}
const ProductCard = ({ product }: { product: Product }) => (
  <Card>{product.name} - {formatPriceWithCurrency(product.price, product.currency)}</Card>
);
const OrderCard = ({ order }: { order: Order }) => (
  <Card>#{order.code} - {order.status}</Card>
);
```

### Shared step stays shared; the branching moves out

```ts
// BAD: one function validates for both flows, branching on `mode`
function validateAccount(input: AccountInput, mode: 'signup' | 'update') {
  if (!input.email.includes('@')) return 'Invalid email';
  if (mode === 'signup' && input.password.length < 8) return 'Password too short';
  if (mode === 'update' && !input.id) return 'Missing id';
  return null;
}

// GOOD: the shared rule is extracted, each flow composes what it needs
const validateEmail = (email: string) => (email.includes('@') ? null : 'Invalid email');

function validateSignup(input: SignupInput) {
  return validateEmail(input.email) ?? (input.password.length < 8 ? 'Password too short' : null);
}
function validateAccountUpdate(input: UpdateInput) {
  return validateEmail(input.email) ?? (input.id ? null : 'Missing id');
}
```

### Branching on data is fine

```ts
// GOOD: callers pass data; the logic is the same for everyone
const orderStatusMeta = {
  [OrderStatus.Pending]: { title: 'Pending' },
  [OrderStatus.Paid]: { title: 'Paid' },
} satisfies Record<OrderStatus, OrderStatusMeta>;

const getStatusTitle = (status: OrderStatus) => orderStatusMeta[status].title;
```

### Branching that only dispatches is fine

```ts
// BAD: the switch holds all the logic, so every product type's rules live in one body
function calculateShippingFee(product: Product) {
  switch (product.type) {
    case ProductType.Physical: {
      const base = product.weightKg * 2;
      return product.fragile ? base + 5 : base;
    }
    case ProductType.Digital:
      return 0;
    case ProductType.Subscription:
      return product.billingCycle === 'yearly' ? 0 : 1.5;
  }
}

// GOOD: the switch only dispatches; each branch is its own focused function
const calculatePhysicalShippingFee = (product: PhysicalProduct) =>
  product.weightKg * 2 + (product.fragile ? 5 : 0);
const calculateDigitalShippingFee = () => 0;
const calculateSubscriptionShippingFee = (product: SubscriptionProduct) =>
  product.billingCycle === 'yearly' ? 0 : 1.5;

function calculateShippingFee(product: Product): number {
  switch (product.type) {
    case ProductType.Physical:
      return calculatePhysicalShippingFee(product);
    case ProductType.Digital:
      return calculateDigitalShippingFee();
    case ProductType.Subscription:
      return calculateSubscriptionShippingFee(product);
    default: {
      const _exhaustive: never = product;
      return _exhaustive;
    }
  }
}
```
