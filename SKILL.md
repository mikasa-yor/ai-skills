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
