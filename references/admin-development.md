# Admin Development

Admin interfaces should optimize for predictable behavior, consistency, maintainability, and semantic reuse.

These are default conventions when product requirements do not specify otherwise.

## 1. Discover before creating

Before implementing an admin feature:

1. Find an existing resource with similar behavior.
2. Identify established table, form, filter, mutation, feedback, permission, and display patterns.
3. Reuse existing components, utilities, and conventions when they represent the same semantic concept.
4. Introduce a new pattern only when existing patterns genuinely do not fit.
5. Keep new abstractions narrowly scoped and clearly owned.

Do not create resource-specific patterns that conflict with an existing reusable semantic pattern.

## 2. Resource / CRUD structure

Think about an admin resource as a coherent domain:

```text
Resource
├── List
│   ├── Search / Filter
│   ├── Table
│   ├── Pagination
│   └── Bulk Actions
├── Create
├── Detail
├── Edit
└── Delete / Other Actions
```

Keep domain rules and display semantics shared across these surfaces when they represent the same concept.

Create and Edit should share form and field logic when their semantics are genuinely the same. Do not force one giant form abstraction when the workflows differ materially.

## 3. Search and filters

Search/filter forms should normally provide:
- Search or Apply;
- Reset;
- predictable initial values;
- predictable reset behavior;
- clear loading/pending behavior.

When filters affect paginated results, changing filters should normally reset pagination to the first page.

Consider URL query parameters as state when filters, search, or pagination need to be shareable, bookmarkable, restorable after refresh, or navigable with browser history. Follow the project's existing convention rather than introducing URL state mechanically.

## 4. Tables

Tables should use consistent conventions for:
- column meaning and naming;
- status/type display;
- empty values;
- dates/times;
- numbers, currency, and percentages;
- long text;
- row actions;
- loading;
- empty results;
- errors;
- pagination.

### Enum/status/type display

Never render raw enum values directly when users need a human-readable representation.

Define display metadata once:

```ts
const orderStatusMeta = {
  [OrderStatus.Pending]: {
    title: 'Pending',
  },
  [OrderStatus.Paid]: {
    title: 'Paid',
  },
} satisfies Record<OrderStatus, OrderStatusMeta>;
```

The metadata may also include icons, tones, colors, or other presentation information when the design system requires them.

Reuse the same semantic mapping in:
- tables;
- search/filter controls;
- create/edit forms;
- detail views;
- badges and status components.

If a label or display semantic changes, there should normally be one place to change it.

## 5. Shared selector/input components

Status, type, category, and similar domain selectors used in multiple forms should normally be separate reusable components.

For example, an `OrderStatusSelect` should be reusable across:
- search/filter forms;
- create forms;
- edit forms;
- other relevant forms.

Do not duplicate enum-to-option mappings across forms.

Wrapper components should forward appropriate underlying input/select props so callers can configure the inner component without the wrapper unnecessarily redefining everything.

## 6. Forms

Forms should have predictable:
- field semantics;
- validation;
- default values;
- submit/pending state;
- server error handling;
- disabled/read-only behavior;
- reset behavior.

Reuse field components when the semantic meaning is the same.

Do not duplicate domain validation or option metadata across create, edit, and filter forms when they share the same rule.

## 7. Date and time

Use a consistent date/time display format across admin tables and related views by default.

Centralize the formatting convention so changing the project-wide display does not require editing many tables.

A feature may override the default when its product requirement explicitly requires a different representation.

## 8. CRUD mutations

Keep create, update, delete, and other mutations consistent across resources:

- show clear pending state;
- prevent accidental duplicate submissions where appropriate;
- provide meaningful success/error feedback according to project conventions;
- use the established React Query cache invalidation/update strategy;
- keep mutation ownership clear;
- do not create duplicated client-side copies of server state.

## 9. Loading, empty, and error states

Admin pages should deliberately handle:
- initial loading;
- background refreshing when relevant;
- empty results;
- filtered empty results;
- server errors;
- permission/authorization failures when applicable.

Do not leave users with an unexplained blank table.

## 10. Destructive actions

Use the project's established confirmation pattern for destructive or difficult-to-reverse actions.

Make the target of the action clear and avoid accidental double submission.

## 11. Bulk actions

When bulk actions exist:
- make selection state explicit;
- clearly communicate how many items are selected;
- use consistent confirmation/feedback patterns;
- refresh or update the authoritative React Query data after completion;
- handle partial failures deliberately if the API can return them.

## 12. Detail views

Detail/read-only views should reuse the same semantic display definitions as tables and forms.

Do not independently recreate status labels, date formats, enum mappings, or domain formatting.

## 13. Permissions

Respect the project's established permission patterns.

Do not assume that hiding or disabling a UI control is sufficient authorization. UI permission handling improves UX; backend authorization remains authoritative.

## 14. Consistency rule

For admin development, consistency across resources is a default design requirement.

When no product requirement specifies otherwise, prefer the established project-wide behavior for:
- search/reset;
- pagination;
- status/type display;
- date/time formatting;
- mutation feedback;
- confirmation dialogs;
- empty/error/loading states;
- table actions;
- form behavior.

Do not introduce a one-off interaction pattern without a concrete reason.

## Examples

### Enum/status display: define metadata once

```tsx
// BAD: raw enum shown, and labels re-mapped per surface
<td>{order.status}</td>                          // "PAID_PENDING_SHIP"
// OrderTable.tsx
{ status === 'paid' ? 'Paid' : 'Pending' }
// OrderDetail.tsx
{ status === 'paid' ? 'Paid!' : 'Waiting' }      // drifted

// GOOD: one metadata map reused by table, filter, form, detail, badge
const orderStatusMeta = {
  [OrderStatus.Pending]: { title: 'Pending' },
  [OrderStatus.Paid]: { title: 'Paid' },
} satisfies Record<OrderStatus, OrderStatusMeta>;

<td>{orderStatusMeta[order.status].title}</td>
```

### Shared selector component

```tsx
// BAD: enum-to-option mapping duplicated in each form
// OrderFilterForm.tsx
<Select options={[{ value: 'pending', label: 'Pending' }, { value: 'paid', label: 'Paid' }]} />
// OrderEditForm.tsx
<Select options={[{ value: 'pending', label: 'Pending' }, { value: 'paid', label: 'Paid' }]} />

// GOOD: one reusable component, forwards underlying Select props
type OrderStatusSelectProps = Omit<React.ComponentProps<typeof Select>, 'options'>;

const OrderStatusSelect = (props: OrderStatusSelectProps) => (
  <Select
    {...props}
    options={Object.values(OrderStatus).map((value) => ({
      value,
      label: orderStatusMeta[value].title,
    }))}
  />
);
```

### Filters reset pagination

```tsx
// BAD: user is on page 5, applies a filter, sees an empty table
const onSearch = (values: Filters) => setFilters(values);

// GOOD: changing filters returns to the first page
const onSearch = (values: Filters) => {
  setFilters(values);
  setPage(1);
};
```

### Search / Reset form

```tsx
// BAD: no way back to the initial state, no pending feedback
<form onSubmit={onSearch}>
  <Input name="keyword" />
  <button type="submit">Go</button>
</form>

// GOOD: Search + Reset, predictable initial values, pending state
<form onSubmit={onSearch}>
  <Input name="keyword" defaultValue={initialFilters.keyword} />
  <button type="submit" disabled={isFetching}>Search</button>
  <button type="button" onClick={() => onReset(initialFilters)}>Reset</button>
</form>
```

### Dates: centralize the format

```tsx
// BAD: each table formats dates its own way
<td>{new Date(o.createdAt).toLocaleDateString()}</td>
<td>{dayjs(u.joinedAt).format('DD/MM/YY')}</td>

// GOOD: one project-wide formatter
// utils/formatDateTime.ts
export const formatDateTime = (value: string | Date) => dayjs(value).format('YYYY-MM-DD HH:mm');

<td>{formatDateTime(o.createdAt)}</td>
<td>{formatDateTime(u.joinedAt)}</td>
```

### Loading, empty, and error states

```tsx
// BAD: unexplained blank table
return <Table rows={data ?? []} />;

// GOOD: every state is deliberate
if (isLoading) return <TableSkeleton />;
if (isError) return <ErrorState onRetry={refetch} />;
if (data.length === 0) {
  return hasActiveFilters
    ? <EmptyState title="No results match your filters" action={<button onClick={reset}>Reset filters</button>} />
    : <EmptyState title="No orders yet" />;
}
return <Table rows={data} />;
```

### Mutations: pending state and server state ownership

```tsx
// BAD: double-submit possible, manual copy of server state, no feedback
const onSave = async () => {
  const updated = await updateOrder(values);
  setOrders((prev) => prev.map((o) => (o.id === updated.id ? updated : o)));
};
<button onClick={onSave}>Save</button>

// GOOD: pending disables the button, cache invalidated, user gets feedback
const queryClient = useQueryClient();
const { mutate, isPending } = useMutation({
  mutationFn: updateOrder,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['orders'] });
    toast.success('Order updated');
  },
  onError: () => toast.error('Failed to update order'),
});
<button onClick={() => mutate(values)} disabled={isPending}>Save</button>
```

### Destructive actions

```tsx
// BAD: immediate delete, ambiguous target
<button onClick={() => deleteOrder(order.id)}>Delete</button>

// GOOD: project's confirmation pattern, names the target, blocks double submit
<ConfirmDialog
  title={`Delete order #${order.code}?`}
  description="This action cannot be undone."
  confirmLabel="Delete"
  isPending={isDeleting}
  onConfirm={() => deleteOrder(order.id)}
/>
```

### Bulk actions

```tsx
// BAD: selection is invisible, partial failures are ignored
<button onClick={() => bulkDelete(selectedIds)}>Delete</button>

// GOOD: explicit count, confirmation, and deliberate partial-failure handling
<button disabled={selectedIds.length === 0} onClick={openConfirm}>
  Delete {selectedIds.length} selected
</button>

// after completion
const { succeeded, failed } = await bulkDelete(selectedIds);
queryClient.invalidateQueries({ queryKey: ['orders'] });
if (failed.length > 0) {
  toast.warning(`${succeeded.length} deleted, ${failed.length} failed`);
}
```

### Permissions

```tsx
// BAD: assumes hiding the button is authorization
{canDelete && <button onClick={() => deleteOrder(id)}>Delete</button>}
// ...while the API endpoint has no permission check

// GOOD: UI hides for UX, backend still enforces authorization
// Frontend: hide/disable the button via the project's permission helper.
// Backend:  DELETE /orders/:id verifies the caller's permission regardless of the UI.
// Frontend also handles the 403 response gracefully (e.g. toast + no cache change).
```

### Detail views reuse display semantics

```tsx
// BAD: detail page re-implements formatting
<span>{order.status === 'paid' ? 'Paid' : 'Pending'}</span>
<span>{new Date(order.createdAt).toLocaleString()}</span>

// GOOD: same metadata and formatter as the table
<span>{orderStatusMeta[order.status].title}</span>
<span>{formatDateTime(order.createdAt)}</span>
```

### Discover before creating (consistency rule)

```text
BAD:  Build the Coupons page with its own "Clear" button, a different
      date format, and a bespoke delete modal.

GOOD: Open the Orders page, copy its Search/Reset behavior, date formatter,
      status badge, and ConfirmDialog usage. Deviate only for a concrete
      product requirement.
```
