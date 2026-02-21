<!-- HATCH3R:BEGIN -->

## Hatch3r Rules

### hatch3r-tooling-hierarchy

Priority order for tools and knowledge sources

# Tooling Hierarchy

## A. GitHub MCP-First (when available)

**Prefer GitHub MCP tools over `gh` CLI** when the MCP server provides typed tools with structured input/output. Use them as the primary interface for GitHub operations.

**Fallback to `gh` CLI only when:**
- The MCP tool catalog lacks the specific capability.
- An MCP call fails repeatedly and the CLI provides a viable alternative.

**Never** use `gh` CLI for operations that have a direct MCP equivalent (issue CRUD, PR CRUD, search, labels).

## B. Documentation MCP for Library Documentation

Use documentation MCP (e.g., Context7) to retrieve up-to-date, version-specific documentation for external libraries and frameworks. This prevents hallucinated APIs and outdated patterns.

**When to use:**
- Working with any external dependency.
- Verifying API signatures, configuration options, or migration paths.
- Reviewing code that uses third-party libraries.
- Writing tests with external test frameworks.
- Debugging errors from external libraries.

**When NOT to use:**
- Internal project specs — use project docs.
- Internal codebase patterns — use Grep, SemanticSearch, or exploration tools.
- General programming concepts not tied to a specific library.

## C. Web Research for External Context

Use web search to retrieve current, real-world information not available in project docs or library documentation.

**When to use:**
- Latest security advisories, CVEs, or vulnerability disclosures for dependencies.
- Breaking changes or deprecations in upcoming dependency versions.
- Current best practices for architecture patterns, deployment strategies, or tooling.
- Novel problems with no match in docs (e.g., obscure error messages, platform-specific quirks).
- Comparing alternative approaches or tools with current community consensus.

**When NOT to use:**
- Questions answerable from project specs or codebase exploration.
- Standard library API questions (use documentation MCP instead).
- Internal project decisions (use project ADRs).

## D. Knowledge Augmentation Priority

When seeking information, follow this priority order:

1. **Project specs and ADRs** — authoritative for project-specific behavior, constraints, and decisions.
2. **Codebase exploration** (Grep, SemanticSearch) — ground truth for current implementation.
3. **Documentation MCP** — authoritative for external library/framework APIs and patterns.
4. **Web research** — current events, best practices, security advisories, novel problems.

Combine sources when valuable: read the spec first, then verify external API usage with docs MCP, then check for recent advisories via web research.

### hatch3r-testing

Test standards and conventions for the project

# Testing Standards

- Unit tests: project test runner. Integration: test runner + emulators/mocks. E2E: browser automation.
- **Deterministic.** Mock time where needed. No wall clock dependency.
- **Isolated.** Each test sets up and tears down its own state.
- **Fast.** Unit tests < 50ms. Integration tests < 2s.
- **Named clearly.** Describe behavior: `"should award 15 XP for 25-min focus block"`.
- **Regression.** Every bug fix includes a test that fails before the fix and passes after.
- **No network.** Unit tests must not make network calls. Use mocks.
- No `any` types in tests. No `.skip` without a linked issue.
- Write tests to `tests/unit/`, `tests/integration/`, `tests/e2e/`, or equivalent.
- Use test fixtures from `tests/fixtures/` or equivalent.

### hatch3r-performance-budgets

Performance budgets and targets for the project

# Performance Budgets

| Metric                            | Budget               |
| --------------------------------- | -------------------- |
| UI render                         | 60fps (16ms/frame)   |
| Cold start to interactive         | 1.5 seconds          |
| Idle CPU usage                    | 1%                   |
| Memory footprint                  | 30 MB                |
| Event processing latency          | 10ms per event       |
| Bundle size (initial, gzipped)    | 500 KB               |
| Backend reads per session start   | ≤ 5 documents        |
| Serverless warm execution         | 500ms                |

### hatch3r-observability

Logging, metrics, and tracing conventions for the project

# Observability

- Use structured JSON logging. No `console.log` in production code.
- Log levels: `error` (failures), `warn` (degraded), `info` (state changes), `debug` (dev only).
- Every log entry includes `correlationId` and `userId` (if available).
- Never log secrets, PII, tokens, passwords, or sensitive content.
- Instrument key operations with timing metrics. Serverless functions log execution time and outcome.
- Client-side: log errors to a sink (e.g., error reporting service), not just `console.error`.
- Prefer event-based metrics over polling. Trace user flows end-to-end with `correlationId`.
- Respect performance budgets: logging must not add > 10ms latency to hot paths.

### hatch3r-migrations

Database migration and schema change patterns for the project

# Migrations

- Schema changes must be backward-compatible. Add fields with defaults; never remove or rename without migration.
- Migration scripts live in a dedicated `migrations/` directory. One script per migration.
- Every migration is idempotent. Safe to re-run. Use document version or `migratedAt` to skip.
- Migration functions log progress and are resumable. Handle partial failures.
- Test migrations against emulator or staging before deploying. Verify data integrity.
- Order: deploy new code (handles old + new schema) → run migration → remove old schema handling.
- Document schema changes in project data model spec.
- Rollback plan required for every migration. Never run destructive migrations without backup verification.
- Hot documents must stay within size limits after migration.

### hatch3r-feature-flags

Feature flag patterns and lifecycle for the project

# Feature Flags

- Use flags for gradual rollout of user-facing changes. Not for A/B experiments without tracking.
- Naming: `FF_{AREA}_{FEATURE}` (e.g., `FF_PET_NEW_CELEBRATION`).
- Store flags in remote config or user document in your backend.
- Client: evaluate with composable/hook. Server: evaluate before processing.
- Every flag has an owner and cleanup deadline (max 30 days after full rollout).
- Remove flag code when feature is fully rolled out. No dead branches.
- Default to disabled. Safe fallback when flag evaluation fails.
- Flags must not gate security or privacy features. Those are always on.
- Document active flags in a tracking table (e.g., `.cursor/rules` or project specs).

### hatch3r-error-handling

Error handling patterns and conventions for the project

# Error Handling

- Define a structured error hierarchy: base error class with `code`, `message`, `cause`.
- Never swallow errors silently. Always re-throw or log with context.
- Use typed error codes (not raw strings). Reference codes in project glossary when applicable.
- User-facing errors are separate from internal errors. Never expose internal details to clients.
- Retry with exponential backoff for transient failures (network, rate limits). Honor `Retry-After` on 429.
- API endpoints return structured error responses `{ code, message, details? }`. Never return stack traces.
- Use framework error boundaries for UI components. Surface user-friendly fallback UI.
- Include `correlationId` in all error logs for tracing across client and server.
- Security: no secrets, tokens, or PII in error messages or logs.

### hatch3r-dependency-management

Rules for managing npm dependencies in the project

# Dependency Management

- Always commit `package-lock.json`. Never use `npm install --no-save`.
- Justify new dependencies in PR description: what it does, why needed, alternatives considered, bundle size impact.
- Prefer well-maintained packages: recent commits, active issues, no known CVEs.
- Pin exact versions for production deps. Use `npm ci` in CI.
- Run `npm audit` before merging dependency changes. Fix high/critical before merge.
- No duplicate packages serving the same purpose. Consolidate on one.
- Remove unused dependencies on every cleanup pass.
- Security patches (CVEs) are P0/P1 priority. Patch within 48h for critical.
- Check bundle size impact against budget. Reject deps that exceed.

### hatch3r-component-conventions

Component conventions for accessibility, design tokens, and responsive patterns

# Component Conventions

## Structure

- Use framework-recommended component syntax (e.g., `<script setup>` for Vue).
- Define props with typed interfaces.
- Define emits/events with typed interfaces.
- Use composables/hooks from project composables directory — never mixins.
- Use stores or equivalent for shared state.

## Naming

- Component files: PascalCase (e.g., `PetWidget.vue`, `QuestPanel.vue`).
- Component names match file names.
- Props: camelCase. Events: kebab-case.

## Styling

- Use the project's design tokens for colors, spacing, typography.
- Prefer utility classes or scoped CSS with BEM naming.
- No inline styles except for dynamic values that can't be expressed as classes.
- No hardcoded color values — use tokens.

## Accessibility (Required)

- All animations: wrap in `prefers-reduced-motion` media query AND check user's `reducedMotion` setting.
- Color contrast: ≥ 4.5:1 for text (WCAG AA).
- Interactive elements: keyboard focusable with visible focus indicator.
- Dynamic content changes: use `aria-live` regions.
- Support high contrast mode.

## Performance

- UI must render at 60fps (≤ 16ms per frame).
- Prefer CSS animations/transitions over JavaScript animations.
- Use conditional visibility (e.g., `v-show`) over conditional mount (e.g., `v-if`) for frequently toggled elements.
- Lazy-load panels that aren't immediately visible.

## Testing

- Snapshot tests for all visual states.
- Component tests for interactive behavior.
- Verify reduced-motion behavior in tests.

### hatch3r-code-standards

TypeScript and file naming conventions for the project

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

### hatch3r-api-design

API endpoint and contract design patterns for the project

# API Design

- Use consistent request/response schemas. Document in spec files.
- All endpoints versioned (URL prefix or header). Backward-compatible changes only.
- Additive changes only: add fields, never rename or remove without migration.
- Error responses follow standard shape: `{ code, message, details? }`.
- Idempotency keys for mutation endpoints. Reject duplicate requests.
- Request validation at the boundary: validate and sanitize before processing.
- List endpoints return envelopes: `{ data, pagination }`. Never raw arrays.
- Rate limiting enforced server-side. Return 429 with `Retry-After` when exceeded.
- API contracts documented in project specs. Update on schema changes.


<!-- HATCH3R:END -->