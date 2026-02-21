---
name: hatch3r-bug-fix
description: Step-by-step bug fix workflow. Diagnose root cause, implement minimal fix, write regression test. Use when fixing bugs, working on bug report issues, or when the user mentions a bug.
---

# Bug Fix Workflow

## Quick Start

```
Task Progress:
- [ ] Step 1: Read the issue and relevant specs
- [ ] Step 2: Produce a diagnosis plan
- [ ] Step 3: Implement minimal fix
- [ ] Step 4: Write regression test
- [ ] Step 5: Verify all tests pass
- [ ] Step 6: Open PR
```

## Step 1: Read Inputs

- Parse the issue body: problem description, reproduction steps, expected/actual behavior, severity, affected area.
- Read relevant project documentation based on affected area (see spec mapping in project context).
- Review existing tests in the affected area.
- For external library docs and current best practices, follow the project's tooling hierarchy.

## Step 2: Diagnosis Plan

Before fixing, output:

- **Root cause hypothesis:** what is wrong and why
- **Files to investigate:** list of files
- **Reproduction strategy:** how to confirm the bug via tests
- **Risks:** what could go wrong with the fix

## Step 3: Minimal Fix

- Fix the root cause with minimal changes.
- Do not refactor unrelated code.
- Do not introduce new dependencies unless absolutely necessary.
- Remove dead code created by the fix.

## Step 4: Regression Test

- Write a test that **fails before** the fix and **passes after**.
- Add edge case tests if the bug reveals coverage gaps.
- Ensure all existing tests still pass.

## Step 5: Verify

```bash
npm run lint && npm run typecheck && npm run test
```

## Step 6: Open PR

Use the project's PR template. Include:

- Root cause explanation
- Fix description with before/after behavior
- Test evidence
- Rollback plan (required for P0/P1)

## Definition of Done

- [ ] Root cause identified and documented in PR
- [ ] Fix implemented with minimal diff
- [ ] Regression test written
- [ ] All existing tests pass
- [ ] No new linter warnings
- [ ] Performance budgets maintained
- [ ] Security/privacy invariants respected
- [ ] If P0/P1: rollback plan documented
