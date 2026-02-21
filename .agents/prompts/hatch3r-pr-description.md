---
id: hatch3r-pr-description
type: prompt
description: Generate a pull request description from staged changes
---

# PR Description

Generate a pull request description for the current changes.

## Instructions

1. Analyze the staged diff and recent commits on this branch
2. Identify the type of change: feature, fix, refactor, docs, infra
3. Write a concise title following: `{type}: {short description}`
4. Write a body with:
   - **Summary** — 1-3 bullet points explaining what changed and why
   - **Test plan** — how to verify the changes work correctly
   - **Breaking changes** — if any, describe the impact and migration path

## Output

A ready-to-use PR title and body in markdown.
