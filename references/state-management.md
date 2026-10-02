# State Management

## Ownership hierarchy

Use the smallest appropriate owner:

1. Derived computation when the value can be calculated.
2. Local React state for component-local interactive state.
3. Jotai for genuinely shared client-side state.
4. React Query for server state and server-cache ownership.

Do not move state to a broader scope merely for convenience.

## Single source of truth

The single-source-of-truth rule applies equally to local React state, Jotai, and React Query.

Never maintain two independent stores containing the same authoritative information unless there is a deliberate architectural reason.

Avoid:

```text
React Query data
      ↓
useEffect
      ↓
Jotai atom
```

when the atom merely mirrors the query.

Prefer:

```text
React Query
    ↓
selector / derived computation
```

or:

```text
Jotai source atom
    ↓
derived atom
```

or:

```text
local source state
    ↓
pure derived value / useMemo
```

Only the authoritative source should be mutated.

## React Query

- React Query owns server state.
- Prefer React Query for API/server data rather than manually managing server state with `useEffect` + `useState`.
- Do not copy query data into Jotai or local state merely for easier access.
- Prefer React Query selectors or transformations for derived server data.
- Keep the query/cache as the authoritative source for server data.

## Jotai

- Jotai owns genuinely shared client state.
- Prefer local state before introducing a shared atom.
- Do not create atoms for data that belongs to React Query.
- Prefer derived atoms/selectors over writable mirrors of other atoms.
- Keep writable atoms as authoritative owners only of the client state they actually own.

## Mutations

Mutate the authoritative source. After server mutations, use the project's established React Query invalidation or cache-update strategy rather than maintaining a second manually synchronized cache.
