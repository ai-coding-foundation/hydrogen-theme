---
id: hatch3r-api-design
type: rule
description: API endpoint and contract design patterns for the project
scope: always
---
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
