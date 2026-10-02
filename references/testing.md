# Testing

Keep testing practical, simple, and behavior-focused.

## Core rules

- Test behavior and contracts rather than implementation details.
- Never test only the happy path.
- Think about meaningful edge cases in the same way you would reason through a programming problem.
- Common candidates include empty arrays, empty strings, zero, boundaries, invalid input, duplicates, missing optional values, and realistic combinations of states.
- Do not mechanically test impossible states when the domain makes them unreachable or when the added complexity provides little confidence.
- Add regression tests for meaningful bug fixes.
- Avoid flaky tests, arbitrary sleeps, and implementation-dependent timing.
- Treat coverage as a signal, not the definition of test quality.

Edge cases should be selected semantically: test cases that could realistically expose incorrect behavior, not every theoretically imaginable input.

## Examples

### Test behavior, not implementation

```tsx
// BAD: asserts internals; breaks on refactor, proves nothing about the user outcome
expect(component.state.isOpen).toBe(true);
expect(setOpenSpy).toHaveBeenCalledTimes(1);

// GOOD: asserts what the user observes
await user.click(screen.getByRole('button', { name: 'Open menu' }));
expect(screen.getByRole('menu')).toBeVisible();
```

### Don't test only the happy path

```ts
// BAD: one happy-path case
test('calculates average', () => {
  expect(average([2, 4, 6])).toBe(4);
});

// GOOD: cases that could realistically expose wrong behavior
describe('average', () => {
  test('averages numbers', () => {
    expect(average([2, 4, 6])).toBe(4);
  });
  test('returns 0 for an empty array', () => {
    expect(average([])).toBe(0);
  });
  test('handles a single element', () => {
    expect(average([5])).toBe(5);
  });
  test('handles negative numbers', () => {
    expect(average([-2, 2])).toBe(0);
  });
});
```

### Pick edge cases semantically

```ts
// BAD: tests an unreachable state because the type system/domain forbids it
test('quantity is a string', () => {
  // quantity is always a number from a validated form
  expect(() => total({ price: 10, quantity: 'abc' as any })).toThrow();
});

// GOOD: tests a reachable boundary
test('quantity of 0 yields a total of 0', () => {
  expect(total({ price: 10, quantity: 0 })).toBe(0);
});
```

### Regression test for a bug fix

```ts
// GOOD: pins the exact bug so it cannot silently return
// Bug: filtering by status did not reset pagination, leaving users on an empty page 3.
test('changing the status filter resets to page 1', async () => {
  render(<OrderList />);
  await goToPage(3);
  await selectStatus('Paid');
  expect(currentPage()).toBe(1);
});
```

### No arbitrary sleeps

```ts
// BAD: flaky timing assumption
await user.click(saveButton);
await new Promise((r) => setTimeout(r, 2000));
expect(screen.getByText('Saved')).toBeInTheDocument();

// GOOD: wait for the observable condition
await user.click(saveButton);
expect(await screen.findByText('Saved')).toBeInTheDocument();
```
