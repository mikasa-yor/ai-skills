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
