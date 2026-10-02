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

## Examples

### Derived state: compute, don't mirror

```tsx
// BAD: useEffect + useState to derive a value
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);

// GOOD: compute directly
const fullName = `${firstName} ${lastName}`;
```

```tsx
// BAD: filtered list kept as separate state, extra render, can go stale
const [visible, setVisible] = useState<Item[]>([]);
useEffect(() => {
  setVisible(items.filter((i) => i.active));
}, [items]);

// GOOD: derived from the source; useMemo names a meaningful computation
const activeItems = useMemo(() => items.filter((i) => i.active), [items]);
```

### Effects are for external systems

```tsx
// BAD: fetching server data with effect + state
const [user, setUser] = useState<User>();
useEffect(() => {
  fetchUser(id).then(setUser);
}, [id]);

// GOOD: React Query owns server state
const { data: user } = useQuery({ queryKey: ['user', id], queryFn: () => fetchUser(id) });
```

```tsx
// GOOD: effect synchronizes with a browser API and cleans up
useEffect(() => {
  const onResize = () => setWidth(window.innerWidth);
  window.addEventListener('resize', onResize);
  return () => window.removeEventListener('resize', onResize);
}, []);
```

### Dependencies must be honest

```tsx
// BAD: lint suppressed to change when the effect runs
useEffect(() => {
  track(pageId, userId);
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, [pageId]);

// GOOD: deps match what the effect uses
useEffect(() => {
  track(pageId, userId);
}, [pageId, userId]);
```

### Cleanup abortable work

```tsx
// BAD: stale response can overwrite newer one
useEffect(() => {
  fetch(`/api/search?q=${query}`).then((r) => r.json()).then(setResults);
}, [query]);

// GOOD: abort work owned by the effect
useEffect(() => {
  const controller = new AbortController();
  fetch(`/api/search?q=${query}`, { signal: controller.signal })
    .then((r) => r.json())
    .then(setResults)
    .catch((e) => {
      if (e.name !== 'AbortError') throw e;
    });
  return () => controller.abort();
}, [query]);
```

### useMemo: meaningful vs trivial

```tsx
// BAD: trivial expression memoized for no reason
const total = useMemo(() => price * quantity, [price, quantity]);

// GOOD: plain expression
const total = price * quantity;

// GOOD: multi-step computation grouped under an intention-revealing name
const overdueInvoiceTotal = useMemo(
  () =>
    invoices
      .filter((i) => i.dueDate < today && !i.paid)
      .reduce((sum, i) => sum + i.amount, 0),
  [invoices, today],
);
```

### useCallback: only when identity matters

```tsx
// BAD: mechanical, nothing depends on identity
const handleClick = useCallback(() => setOpen(true), []);
return <button onClick={handleClick} />;

// GOOD: plain handler
return <button onClick={() => setOpen(true)} />;

// GOOD: memoized child relies on stable identity
const handleSelect = useCallback((id: string) => select(id), [select]);
return <MemoizedList onSelect={handleSelect} />;
```

### Render purity

```tsx
// BAD: side effect during render
function Counter({ id }: { id: string }) {
  analytics.track('viewed', id);
  return <div>{id}</div>;
}

// GOOD: side effect in an effect (or event handler)
function Counter({ id }: { id: string }) {
  useEffect(() => {
    analytics.track('viewed', id);
  }, [id]);
  return <div>{id}</div>;
}
```

### Refs are not data flow

```tsx
// BAD: ref used to pass data between renders/components
const selectedIdRef = useRef<string>();
const onSelect = (id: string) => {
  selectedIdRef.current = id; // UI won't update
};

// GOOD: state for data, ref for DOM
const [selectedId, setSelectedId] = useState<string>();
const inputRef = useRef<HTMLInputElement>(null);
inputRef.current?.focus();
```

### Wrapper components forward props

```tsx
// BAD: redefines the prop surface, drops everything else
type Props = { value: string; onChange: (v: string) => void; placeholder?: string };
const SearchInput = ({ value, onChange, placeholder }: Props) => (
  <Input value={value} onChange={(e) => onChange(e.target.value)} placeholder={placeholder} />
);

// GOOD: forward underlying props
type SearchInputProps = React.ComponentProps<typeof Input>;
const SearchInput = (props: SearchInputProps) => <Input {...props} />;
```
