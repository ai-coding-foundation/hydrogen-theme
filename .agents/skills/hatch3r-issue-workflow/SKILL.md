---
name: hatch3r-issue-workflow
description: Guides the 8-step agentic development workflow for GitHub issues. Covers parsing issues, loading skills, reading specs, planning, implementing, testing, opening PRs, and addressing review. Use when working on any GitHub issue or when the user mentions an issue number.
---

# Issue Workflow

## Quick Start

When assigned a GitHub issue, follow these 8 steps in order:

```
Task Progress:
- [ ] Step 1: Parse the issue
- [ ] Step 2: Load the issue-type skill
- [ ] Step 3: Read relevant specs
- [ ] Step 4: Produce a plan
- [ ] Step 5: Implement
- [ ] Step 6: Test
- [ ] Step 7: Open PR
- [ ] Step 8: Address review
```

## Step 1: Parse the Issue

- Read all fields from the issue template.
- Identify: type (bug/feature/refactor/qa), affected area, priority, acceptance criteria.
- Note linked issues and spec references.

## Step 2: Load the Issue-Type Skill

Navigate to the matching skill based on issue type:

| Issue Type        | Skill                    |
| ----------------- | ------------------------ |
| Bug report        | hatch3r-bug-fix          |
| Feature request   | hatch3r-feature          |
| Code refactor     | hatch3r-refactor         |
| Logical refactor  | hatch3r-logical-refactor |
| Visual refactor   | hatch3r-visual-refactor  |
| QA E2E validation | hatch3r-qa-validation    |

## Step 3: Read Relevant Specs

Load only the specs relevant to the issue area. See the spec mapping table in the project context rule.

Also check project ADRs for architectural constraints.

- For external library docs and current best practices, follow the project's tooling hierarchy.

## Step 4: Produce a Plan

Output a structured plan before writing code:

- **Approach:** strategy
- **Files to touch:** list with brief description per file
- **Tests to write:** list
- **Risks:** what could go wrong
- **Open questions:** if any, propose answers

## Step 4b: Sub-Agent Delegation (Epics Only)

When working on an epic with multiple sub-issues, delegate each sub-issue to a dedicated implementer sub-agent rather than implementing sequentially. The board-pickup command orchestrates this automatically, but if running issue-workflow standalone:

1. **Group sub-issues by dependency level** from the epic's Implementation Order.
2. **Spawn one implementer sub-agent per sub-issue** using the Task tool. Include: issue number, body, acceptance criteria, issue type, parent epic context, and spec references.
3. **Launch sub-issues at the same dependency level in parallel** (max 4 concurrent).
4. **Await all sub-agents at a level** before starting the next level.
5. **Review results** from each sub-agent. Resolve any file conflicts between parallel outputs.

The implementer sub-agent protocol is defined in the hatch3r-implementer agent. Each sub-agent handles its own implementation and testing but does NOT create branches, commits, or PRs.

## Step 5: Implement

- Follow the plan. Use stable IDs from the project glossary.
- Do not expand scope beyond acceptance criteria.
- Remove dead code. Keep changes minimal and focused.

## Step 6: Test

- Unit tests for new logic. Integration tests for cross-module interactions.
- Database/security rules tests if rules changed.
- Regression tests for preserved behavior.
- All tests must be deterministic, isolated, and fast.

## Step 7: Open PR

- Use the project's PR template. Fill every section.
- Link to the issue. Include plan, implementation summary, test evidence.
- Self-review against the Definition of Done from the loaded skill.

## Step 8: Address Review

- Respond to every review comment.
- Push fixes as new commits (don't force-push during review).
- Re-request review after addressing all comments.

## Escalation

Stop and ask when:

1. Acceptance criteria are contradictory.
2. Security concern discovered.
3. Architectural decision needed (new ADR required).
4. Spec conflict found.
5. Scope creep detected.
6. Test infrastructure missing.
