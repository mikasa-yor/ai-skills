# TypeScript

## Type safety

- Prefer strict, precise types.
- Avoid `any`.
- Use `unknown` only at genuine runtime/trust boundaries, then narrow it before use.
- Avoid `as` assertions used to bypass type checking. Prefer correct source types, narrowing, validation, or type guards.
- A narrowly scoped assertion is acceptable only when the relationship is known to be safe and TypeScript cannot express it naturally.
- Never use `as unknown as SomeType` simply to force compilation.
- Avoid non-null assertions used only to suppress compiler errors.

## Domain modeling

- Prefer discriminated unions for mutually exclusive variants.
- Do not use a generic catch-all object containing optional fields for every possible variant.
- Prefer narrow domain types over broad `string`, `number`, or object shapes when real constraints exist.
- Use exhaustive checks for discriminated unions when all variants must be handled.
- Prefer type inference when it remains precise; use explicit types for meaningful contracts.
- Handle nullability explicitly.

## Single source of truth

Reuse authoritative type definitions rather than duplicating the same domain concept.

Reuse a type, enum, constant, function, or other definition when it represents the same semantic concept or business rule. Do not force unrelated concepts to share a type merely because their current fields happen to match.

## Runtime boundaries

TypeScript types do not validate runtime data. Treat API responses, parsed JSON, storage values, and other external inputs as runtime boundaries and validate or narrow them when necessary.
