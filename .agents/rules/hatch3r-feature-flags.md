---
id: hatch3r-feature-flags
type: rule
description: Feature flag patterns and lifecycle for the project
scope: always
---
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
