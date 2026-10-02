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
