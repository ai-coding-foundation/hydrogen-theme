---
id: hatch3r-bug-triage
type: prompt
description: Triage a bug report and suggest investigation steps
---
# Bug Triage

Triage the described bug and produce an investigation plan.

## Instructions

1. Classify severity: P0 (data loss, security), P1 (broken feature), P2 (degraded), P3 (cosmetic)
2. Identify affected area from the description
3. List 3-5 investigation steps with specific files/functions to check
4. Suggest a minimal reproduction path
5. Propose a fix approach if the root cause is evident

## Output

A structured triage report with severity, investigation steps, and suggested fix.
