# TypeScript

## Type safety

- Prefer strict, precise types.
- Avoid `any`.
- Use `unknown` only at genuine runtime/trust boundaries, then narrow it before use.
- Avoid `as` assertions used to bypass type checking. Prefer correct source types, narrowing, validation, or type guards.
- A narrowly scoped assertion is acceptable only when the relationship is known to be safe and TypeScript cannot express it naturally.
- Never use `as unknown as SomeType` simply to force compilation.
- Avoid non-null assertions used only to suppress compiler errors.

## Domain modeling

- Prefer discriminated unions for mutually exclusive variants.
- Do not use a generic catch-all object containing optional fields for every possible variant.
- Prefer narrow domain types over broad `string`, `number`, or object shapes when real constraints exist.
- Use exhaustive checks for discriminated unions when all variants must be handled.
- Prefer type inference when it remains precise; use explicit types for meaningful contracts.
- Handle nullability explicitly.

## Single source of truth

Reuse authoritative type definitions rather than duplicating the same domain concept.

Reuse a type, enum, constant, function, or other definition when it represents the same semantic concept or business rule. Do not force unrelated concepts to share a type merely because their current fields happen to match.

## Runtime boundaries

TypeScript types do not validate runtime data. Treat API responses, parsed JSON, storage values, and other external inputs as runtime boundaries and validate or narrow them when necessary.

## Examples

### Avoid `any` and unchecked assertions at runtime boundaries

```ts
// BAD: `any` and `as` hide the fact that the response is unvalidated
const user = (await res.json()) as User;
const data: any = JSON.parse(raw);

// GOOD: treat input as unknown, then validate/narrow
const data: unknown = JSON.parse(raw);
if (!isUser(data)) {
  throw new Error('Invalid user payload');
}
// data is User here
```

### Never force-compile with `as unknown as`

```ts
// BAD
const order = payload as unknown as Order;

// GOOD: fix the source type or narrow with a type guard
function isOrder(value: unknown): value is Order {
  return typeof value === 'object' && value !== null && 'id' in value && 'status' in value;
}
```

### Non-null assertions

```ts
// BAD: silences the compiler, crashes at runtime if the row is missing
const name = users.find((u) => u.id === id)!.name;

// GOOD: handle nullability explicitly
const user = users.find((u) => u.id === id);
if (!user) {
  throw new Error(`User ${id} not found`);
}
const name = user.name;
```

### Discriminated unions instead of optional-field bags

```ts
// BAD: every combination is "valid" to the compiler
type Payment = {
  method: 'card' | 'bank';
  cardNumber?: string;
  iban?: string;
};

// GOOD: mutually exclusive variants
type Payment =
  | { method: 'card'; cardNumber: string }
  | { method: 'bank'; iban: string };
```

### Exhaustive checks

```ts
// BAD: adding a new variant silently falls through
function label(p: Payment) {
  if (p.method === 'card') return 'Card';
  return 'Bank';
}

// GOOD: compiler errors when a variant is unhandled
function label(p: Payment): string {
  switch (p.method) {
    case 'card':
      return 'Card';
    case 'bank':
      return 'Bank';
    default: {
      const _exhaustive: never = p;
      return _exhaustive;
    }
  }
}
```

### Narrow domain types over broad primitives

```ts
// BAD
function setStatus(status: string) {}

// GOOD
type OrderStatus = 'pending' | 'paid' | 'cancelled';
function setStatus(status: OrderStatus) {}
```

### Single source of truth for types

```ts
// BAD: same concept redefined, will drift
type OrderRowStatus = 'pending' | 'paid' | 'cancelled';
type OrderFilterStatus = 'pending' | 'paid' | 'cancelled';

// GOOD: one authoritative definition, reused
type OrderStatus = 'pending' | 'paid' | 'cancelled';
```
