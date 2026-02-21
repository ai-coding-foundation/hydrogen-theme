---
name: hatch3r-refactor
description: Internal code quality improvement workflow without changing external behavior. Use when refactoring code structure, simplifying modules, or improving maintainability.
---

# Code Refactor Workflow

## Quick Start

```
Task Progress:
- [ ] Step 1: Read the issue, specs, and existing tests
- [ ] Step 2: Produce a refactor plan
- [ ] Step 3: Implement with behavioral preservation
- [ ] Step 4: Verify all tests pass, add regression tests
- [ ] Step 5: Open PR
```

## Step 1: Read Inputs

- Parse the issue body: motivation, proposed change, affected files, safety plan, risk analysis, acceptance criteria.
- Read project quality standards documentation.
- Read specs for the area being refactored.
- Review all existing tests — every one must still pass after refactoring.
- For external library docs and current best practices, follow the project's tooling hierarchy.

## Step 2: Refactor Plan

Before changing code, output:

- **Goal:** what improves (readability, performance, maintainability)
- **Strategy:** how the refactor works
- **Files to modify:** list with what changes
- **Behavioral invariant:** what must NOT change
- **Risk assessment:** what could go wrong, how to detect
- **Rollback:** how to revert if needed

## Step 3: Implement

- Refactor with minimum changes needed.
- Preserve all public interfaces and external behavior.
- Remove dead code created by the refactor.
- Do not introduce new dependencies.
- If a bug is found, document it — fix in a separate PR.

## Step 4: Verify

- All existing tests must pass without modification.
- Add regression tests for previously untested at-risk behavior.
- Performance verification if refactored code is on a hot path.

```bash
npm run lint && npm run typecheck && npm run test
```

## Step 5: Open PR

Use the project's PR template. Include:

- Motivation (why this refactor now)
- Before/after structure (high-level description)
- Proof of behavioral preservation (test results)
- Performance impact (if applicable)

## Definition of Done

- [ ] All existing tests pass without modification
- [ ] Regression tests added for at-risk behavior
- [ ] No new linter warnings
- [ ] Performance budgets maintained
- [ ] Dead code removed
- [ ] No external behavior changed
- [ ] No new dependencies introduced
