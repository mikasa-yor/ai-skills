# Architecture

## Dependency direction

A common top-down structure is:

```text
Pages
  ↓
Components / Hooks
  ↓
APIs
  ↓
Utils / Constants
  ↓
Types
```

The exact folder names may differ by project, but dependency direction should remain clear.

- Higher layers may depend on lower layers.
- Lower layers must not import higher layers.
- Avoid circular dependencies.

## Responsibilities

### Pages
Compose a page or feature and coordinate high-level behavior.

### Components
Render UI and coordinate component-level interaction.

### Hooks
Encapsulate reusable React behavior and state access.

### APIs
Own communication with backend or external services.

### Utils / constants
Contain genuinely reusable lower-level helpers and stable domain constants.

### Types
Contain shared TypeScript contracts when they need a lower-level shared home.

## Before introducing architecture

Discover the existing architecture first:

1. Find nearby implementations solving a similar problem.
2. Identify ownership of state and business rules.
3. Reuse established boundaries when they fit.
4. Change an existing authoritative source instead of creating a second source.
5. Introduce a new abstraction only when its semantic responsibility is clear.
6. Avoid unrelated refactors.

Architecture should make dependency direction and ownership easy to understand.
