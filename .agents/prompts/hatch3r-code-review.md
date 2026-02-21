---
id: hatch3r-code-review
type: prompt
description: Review code changes for quality, security, and correctness
---
# Code Review

Review the provided code changes for quality, security, and correctness.

## Instructions

1. Check correctness: does the code do what it's supposed to?
2. Check security: input validation, auth checks, no secrets in code
3. Check quality: naming, complexity, test coverage, dead code
4. Check performance: N+1 queries, unnecessary allocations, bundle impact
5. Check accessibility: if UI changes, verify keyboard nav and screen reader support

## Output

Feedback organized by priority:
- **Critical** — must fix before merge
- **Warning** — should fix, not blocking
- **Suggestion** — nice to have improvements
