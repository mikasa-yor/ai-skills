---
name: frontend-engineering
description: Maintainability-first frontend engineering conventions for TypeScript and React applications. Load only the relevant reference files for the current task.
---

# Frontend Engineering

## Core philosophy

Optimize for:
1. Long-term maintainability
2. Readability
3. Consistency

Core principle:

> One unique reason for change should normally require changing one place.

Keep a single source of truth for each semantic concept or business rule. Reuse by semantic meaning, not merely because code looks similar. Prefer simple solutions, avoid premature abstraction and overengineering, and discover existing project patterns before introducing new ones.

## Reference routing

Load only the relevant reference when needed:

- `references/typescript.md` — TypeScript type safety and domain modeling.
- `references/clean-code.md` — clean code, immutability, functions, naming, control flow, errors.
- `references/react.md` — components, hooks, effects, memoization, derived state.
- `references/state-management.md` — local state, Jotai, React Query, state ownership.
- `references/architecture.md` — layers and dependency direction.
- `references/testing.md` — practical testing and edge cases.
- `references/admin-development.md` — admin tables, forms, filters, CRUD, display semantics, mutations.

## Before changing code

1. Understand the task and acceptance criteria.
2. Discover nearby existing implementations and project conventions.
3. Identify the authoritative source of each piece of state/business logic.
4. Prefer changing or reusing an existing source over creating a second one.
5. Keep unrelated refactors out of scope.
6. Validate behavior with relevant checks.

## Examples

### One reason for change, one place to change

```ts
// BAD: the same business rule lives in three places
// CartSummary.tsx
const hasFreeShipping = subtotal >= 50;
// CheckoutForm.tsx
const showFreeShippingBanner = subtotal >= 50;
// shippingApi.ts
if (subtotal >= 50) fee = 0;

// GOOD: one authoritative definition; changing the threshold touches one file
// constants/shipping.ts
export const FREE_SHIPPING_THRESHOLD = 50;
export const hasFreeShipping = (subtotal: number) => subtotal >= FREE_SHIPPING_THRESHOLD;
```

### Reuse by meaning, not by look

```ts
// BAD: unrelated concepts forced into one helper because the code looks alike
const isLongText = (s: string) => s.length > 100; // used for both a bio field and a tweet limit

// GOOD: separate rules that can change independently
const isBioTooLong = (bio: string) => bio.length > MAX_BIO_LENGTH;
const isPostTooLong = (post: string) => post.length > MAX_POST_LENGTH;
```

### Simple first, no premature abstraction

```tsx
// BAD: generic framework for a single use case
const createFormFactory = <T,>(config: FormFactoryConfig<T>) => /* 80 lines */;
const ContactForm = createFormFactory({ fields: ['email'], layout: 'single' });

// GOOD: plain component until a second real use case shows what to abstract
function ContactForm() {
  return (
    <form>
      <Input name="email" />
    </form>
  );
}
```

### Before changing code: workflow

```text
Task: "Show the order status as a colored badge in the order detail page."

BAD:
  Write a new <StatusBadge> with its own status-to-color switch inside
  OrderDetail.tsx. While there, rename variables and reformat the file.

GOOD:
  1. Understand: badge on detail page, colors per status.
  2. Discover: the Orders table already renders a status badge.
  3. Authoritative source: `orderStatusMeta` and the existing badge component.
  4. Reuse that badge; extend `orderStatusMeta` only if a color is missing.
  5. Out of scope: no renames, no reformatting, no unrelated refactors.
  6. Validate: run typecheck, lint, and the related tests.
```
