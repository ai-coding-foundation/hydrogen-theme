---
id: hatch3r-code-standards
type: rule
description: TypeScript and file naming conventions for the project
scope: always
---
# Code Standards

- TypeScript strict mode. No `any`. No `@ts-ignore` without a linked issue.
- Functions: `camelCase`. Types/Interfaces: `PascalCase`. Constants: `SCREAMING_SNAKE`.
- Component files: `PascalCase` (match framework convention). Logic files: `camelCase.ts`.
- Max function length: 50 lines. Max file: 400 lines. Cyclomatic complexity: 10.
- Use framework-recommended component patterns (e.g., typed props and emits).
- Use stores or equivalent for shared state. Prefer composables/hooks over mixins.
- Use design tokens for colors, spacing, typography. No ad-hoc styling.
- All animations must respect `prefers-reduced-motion`.
- Run lint and typecheck before committing.
