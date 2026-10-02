# Clean Code

## Maintainability

- Long-term maintainability is the primary goal.
- Keep one source of truth for each semantic concept or business rule.
- Reuse functions, constants, enums, types, classes, or other abstractions when they represent the same meaning.
- Do not abstract code merely because it looks similar.
- Avoid both duplication of the same business rule and premature abstraction.
- Prefer the smallest well-designed change that solves the task.
- Avoid unrelated refactoring and overengineering.

## Immutability

- Prefer `const`.
- Avoid `let` when logic can be expressed through a direct expression or smaller functions returning values.
- Prefer immutable object and array updates.
- Avoid direct mutation, especially destructive operations such as `delete`.
- Mutation is acceptable when it is clearly simpler, controlled, safe, and does not introduce surprising side effects.

## Readability

- Prefer simple, intention-revealing code over clever or compressed code.
- Use early returns and guard clauses to reduce unnecessary nesting.
- Keep functions focused on a coherent responsibility.
- Do not split functions merely because they exceed an arbitrary line count.
- Use names that reveal intent and domain meaning.
- Prefer functions with roughly three or fewer parameters.
- When more parameters are genuinely needed, consider a well-named object parameter if it improves the API.
- Do not create an object parameter mechanically just to satisfy the parameter-count preference.

## Control flow

- Prefer `switch` for multiple discrete values of the same discriminant or enum when it improves clarity, especially for exhaustive handling.
- Use ordinary conditionals when ranges or compound boolean conditions are clearer.
- Use `map` for transformation.
- Use `filter` for selection.
- Use `forEach` for straightforward per-item effects when no result collection is needed.
- Use `for...of` when `break`, `continue`, sequential `await`, or imperative control flow is clearer.
- Use `while` when the problem naturally requires it.
- Do not mechanically replace every loop with an array method.

## Magic values

Name literals when they represent meaningful business rules, limits, statuses, or domain concepts. Do not turn every obvious literal into a constant.

## Side effects

- Prefer pure functions for calculations and transformations when practical.
- Make side effects explicit.
- Keep business logic separate from I/O when doing so improves clarity and testability.

## Errors

- Never silently swallow an error without a deliberate, justified reason.
- If the current layer cannot meaningfully handle an error, propagate it.
- Do not catch an error merely to hide it behind a default value.
- API functions should normally log meaningful failures with `console.error` and rethrow so React Query or the caller can handle the error.
- Check existing HTTP clients and interceptors first to avoid duplicate logging.
- Wrap `JSON.parse` of potentially invalid input in `try/catch`.
- Handle JSON parsing failure intentionally. Return a fallback only when that fallback is part of the expected contract; otherwise propagate or return an explicit error.
- Parsing JSON into a TypeScript generic does not validate the runtime structure.

## Comments

Prefer intention-revealing names and clear structure over comments.

Comments should explain **why**, not **what**.

Use comments for:
- business constraints;
- non-obvious technical constraints;
- rationale behind unusual decisions;
- compatibility or safety considerations.

Do not add comments that merely narrate the next line of code.

## Examples

### Immutability

```ts
// BAD: let + mutation
let label = 'Unknown';
if (status === 'paid') label = 'Paid';
const next = { ...order };
delete next.note;

// GOOD: direct expression, immutable update
const label = status === 'paid' ? 'Paid' : 'Unknown';
const { note, ...next } = order;
```

### Guard clauses

```ts
// BAD: nested
function getDiscount(user?: User) {
  if (user) {
    if (user.isActive) {
      return user.discount;
    }
  }
  return 0;
}

// GOOD: early returns
function getDiscount(user?: User) {
  if (!user?.isActive) return 0;
  return user.discount;
}
```

### Parameters

```ts
// BAD: positional soup
createInvoice(customerId, true, false, 'USD', 30);

// GOOD: named object when more parameters are genuinely needed
createInvoice({ customerId, sendEmail: true, draft: false, currency: 'USD', dueInDays: 30 });
```

### Control flow: pick the construct that fits

```ts
// BAD: forEach used to build a result, flag-based loop
const ids: string[] = [];
items.forEach((i) => ids.push(i.id));

// GOOD: map transforms
const ids = items.map((i) => i.id);

// BAD: map/forEach cannot break or await sequentially
items.forEach(async (i) => await save(i));

// GOOD: for...of for sequential await / break
for (const item of items) {
  await save(item);
}
```

### Magic values

```ts
// BAD: meaning is hidden
if (password.length < 8) {}
setTimeout(refresh, 300000);

// GOOD: name business rules
const MIN_PASSWORD_LENGTH = 8;
const REFRESH_INTERVAL_MS = 5 * 60 * 1000;

// GOOD: obvious literals stay literal
const isFirst = index === 0;
```

### Errors

```ts
// BAD: swallowed, hides failure behind a default
async function getUser(id: string) {
  try {
    return await api.get(`/users/${id}`);
  } catch {
    return null;
  }
}

// GOOD: log meaningfully and rethrow so React Query / caller decides
async function getUser(id: string) {
  try {
    return await api.get(`/users/${id}`);
  } catch (error) {
    console.error('Failed to fetch user', { id, error });
    throw error;
  }
}
```

### JSON parsing

```ts
// BAD: throws on bad input, and the generic validates nothing
const settings = JSON.parse(raw) as Settings;

// GOOD: handle failure intentionally, validate shape
function parseSettings(raw: string): Settings {
  let parsed: unknown;
  try {
    parsed = JSON.parse(raw);
  } catch (error) {
    throw new Error('Settings JSON is malformed', { cause: error });
  }
  if (!isSettings(parsed)) {
    throw new Error('Settings JSON has unexpected shape');
  }
  return parsed;
}
```

### Comments: why, not what

```ts
// BAD: narrates the code
// increment retry count
retries += 1;

// GOOD: explains a non-obvious constraint
// The payment gateway rejects retries within 2s of the previous attempt.
const RETRY_DELAY_MS = 2000;
```

### Reuse by meaning, not by look

```ts
// BAD: merged because the code looks alike, but the rules are unrelated
const clampTo100 = (n: number) => Math.min(n, 100); // used for both discount % and page size

// GOOD: separate concepts, separate rules (they can change independently)
const MAX_DISCOUNT_PERCENT = 100;
const MAX_PAGE_SIZE = 100;
```
