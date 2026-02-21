---
id: hatch3r-testing
type: rule
description: Test standards and conventions for the project
scope: always
---

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
