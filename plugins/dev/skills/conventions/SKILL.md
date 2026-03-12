---
description: Development team coding conventions for Unity/C# and Node.js/TS
globs:
  - "**/*.cs"
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
---

# Dev Conventions

When writing or modifying code, follow these team rules:

## Unity / C#
- Namespaces: PascalCase matching folder structure
- MonoBehaviour fields: `[SerializeField] private` with camelCase
- Coroutines: prefix with `Co_` (e.g., `Co_FadeOut`)
- Avoid `Find` / `GetComponent` in Update loops
- Null-check with pattern matching: `if (obj is not null)`

## Node.js / TypeScript
- Variables/functions: camelCase
- Classes/interfaces: PascalCase
- Constants: UPPER_SNAKE_CASE
- Files: kebab-case (e.g., `user-service.ts`)
- Max 30 lines per function; return early to reduce nesting
- Max 3 parameters; use options object for more

## Git
- Branch naming: `{type}/{ticket}-{description}` (e.g., `feat/GS-123-add-login`)
- Commit message: imperative mood, max 72 chars
- No force push to main/develop