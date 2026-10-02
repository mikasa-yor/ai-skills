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

## Examples

### Don't mirror React Query into Jotai

```tsx
// BAD: two stores for the same authoritative data
const usersAtom = atom<User[]>([]);

function UsersLoader() {
  const { data } = useQuery({ queryKey: ['users'], queryFn: fetchUsers });
  const setUsers = useSetAtom(usersAtom);
  useEffect(() => {
    if (data) setUsers(data);
  }, [data, setUsers]);
  return null;
}

// GOOD: React Query is the source, derive with a selector
const useActiveUsers = () =>
  useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
    select: (users) => users.filter((u) => u.active),
  });
```

### Don't copy query data into local state

```tsx
// BAD: local copy goes stale when the cache updates
const { data } = useQuery({ queryKey: ['user', id], queryFn: () => fetchUser(id) });
const [user, setUser] = useState(data);

// GOOD: read from the query; local state only for what the user is editing
const { data: user } = useQuery({ queryKey: ['user', id], queryFn: () => fetchUser(id) });
const [draftName, setDraftName] = useState<string>();
const displayName = draftName ?? user?.name ?? '';
```

### Derived atoms over writable mirrors

```ts
// BAD: second writable atom kept in sync by hand
const itemsAtom = atom<Item[]>([]);
const itemCountAtom = atom(0);
// ...every place that sets itemsAtom must also remember to set itemCountAtom

// GOOD: derived atom, cannot drift
const itemsAtom = atom<Item[]>([]);
const itemCountAtom = atom((get) => get(itemsAtom).length);
```

### Smallest appropriate owner

```tsx
// BAD: modal open state in a global atom when only one component uses it
const isConfirmOpenAtom = atom(false);

// GOOD: local state
const [isConfirmOpen, setIsConfirmOpen] = useState(false);

// GOOD: atom is justified when several distant components share it
const sidebarCollapsedAtom = atom(false);
```

### Mutations: update the authoritative source

```tsx
// BAD: manually patching a second copy after a mutation
const { mutate } = useMutation({
  mutationFn: updateUser,
  onSuccess: (user) => {
    setUser(user); // local copy
    setUsersAtom((prev) => prev.map((u) => (u.id === user.id ? user : u))); // atom copy
  },
});

// GOOD: use the project's established invalidation (or cache update) strategy
const queryClient = useQueryClient();
const { mutate } = useMutation({
  mutationFn: updateUser,
  onSuccess: () => queryClient.invalidateQueries({ queryKey: ['users'] }),
});
```
