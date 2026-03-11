---
description: Team coding conventions and style guide
globs:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
  - "**/*.cs"
---

# Team Conventions

When writing or modifying code, follow these team rules:

## Naming
- Variables/functions: camelCase
- Classes/components: PascalCase
- Constants: UPPER_SNAKE_CASE
- Files: kebab-case (e.g., `user-service.ts`)

## Functions
- Max 30 lines per function
- Return early to reduce nesting
- Max 3 parameters; use options object for more

## Error Handling
- Never silently catch errors
- Always log or rethrow with context
- Validate inputs at public API boundaries

## Comments
- Write "why", not "what"
- No commented-out code — use git history
- TODO format: `// TODO(TICKET-123): description`
