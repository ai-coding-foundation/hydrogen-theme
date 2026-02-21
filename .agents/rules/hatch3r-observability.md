---
id: hatch3r-observability
type: rule
description: Logging, metrics, and tracing conventions for the project
scope: always
---

# Observability

- Use structured JSON logging. No `console.log` in production code.
- Log levels: `error` (failures), `warn` (degraded), `info` (state changes), `debug` (dev only).
- Every log entry includes `correlationId` and `userId` (if available).
- Never log secrets, PII, tokens, passwords, or sensitive content.
- Instrument key operations with timing metrics. Serverless functions log execution time and outcome.
- Client-side: log errors to a sink (e.g., error reporting service), not just `console.error`.
- Prefer event-based metrics over polling. Trace user flows end-to-end with `correlationId`.
- Respect performance budgets: logging must not add > 10ms latency to hot paths.
