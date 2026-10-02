# React

## Components

- Use function components; do not introduce class components.
- Keep render pure. Never perform side effects during render.
- Keep components focused on one coherent responsibility.
- Around 200 lines is a signal to review responsibility and abstraction, not a hard limit.
- Do not split components solely to reduce line count.
- Simple wrapper components should generally forward the underlying component props instead of unnecessarily redefining the full prop surface.

## Refs

- Do not use refs as normal application data flow.
- Prefer props, state, React Query, Jotai, or derived values for data flow.
- Use refs primarily for DOM interaction or imperative integrations.

## Hooks

- Follow the Rules of Hooks.
- Before adding a hook, ask whether ordinary computation, props, or existing state is sufficient.
- Custom hooks should represent meaningful reusable behavior rather than arbitrary extraction.

### useEffect

Use `useEffect` primarily to synchronize React with external systems such as:
- subscriptions;
- timers;
- browser/DOM APIs;
- external imperative integrations.

Strong rule:

> Do not use `useEffect` + `useState` to calculate derived state from existing props, state, React Query data, or Jotai atoms.

Effect dependencies should reflect the values actually used by the effect. Do not suppress dependency linting merely to change when an effect runs.

Clean up resources owned by the effect, including subscriptions, listeners, timers, and abortable work where applicable.

### Derived state

Never create independent state merely to mirror another source of truth.

If a value can be computed from existing state, props, React Query data, or Jotai state:
- compute it directly when simple;
- use `useMemo` when the computation is meaningful or complex and grouping it under an intention-revealing variable improves the code.

Avoid:

```text
source state -> useEffect -> setDerivedState
```

unless the derived value deliberately becomes independently owned state for a real architectural reason.

### useMemo

This codebase may use `useMemo` both for meaningful memoization and to group non-trivial derived computation under a clear name.

Prefer:

```tsx
const eligibleItems = useMemo(
  () => items.filter(isEligible),
  [items],
);
```

over flattening meaningful computation into JSX.

A loop, multi-step calculation, or computation spanning meaningful logic is a good candidate for `useMemo`.

Do not wrap trivial expressions in `useMemo` without a reason.

### useCallback

Do not use `useCallback` mechanically.

Use it when function identity has a meaningful purpose, such as:
- a memoized child depends on stable callback identity;
- a hook dependency requires stable identity;
- an external integration requires stable identity.
