# ADR-0002: Stage-Based Life-Stage Segmentation as Primary IA

## Purpose

This ADR records the decision to use life-stage segmentation as the primary information architecture for the EVOHERA store, affecting navigation, product organization, content, and marketing.

---

- **Status:** ACCEPTED
- **Date:** 2026-02-20
- **Decision Makers:** TBD

## Context

The store needs a primary navigation and product organization strategy. Women's supplement shoppers face decision fatigue when browsing by ingredient or symptom. The reference experience (Biologica) demonstrates that life-stage segmentation ("I'm 47 → Transition") reduces friction and increases conversion confidence.

## Decision

Use three life-stage segments (Cycle 18-40, Transition 40-55, Thrive 50+) as the primary information architecture across navigation, product pages, content, and marketing.

## Alternatives Considered

| Option | Pros | Cons |
|--------|------|------|
| Category-based (by ingredient type) | Familiar e-commerce pattern, easy to add products | High decision fatigue, doesn't map to user identity |
| Symptom-based (sleep/energy/mood) | Intent-driven, strong SEO | Overlapping symptoms across stages, complex IA |
| Traditional age-range | Simple | Feels clinical, less aspirational |

## Consequences

### Positive

- Reduced decision fatigue
- Clear content taxonomy
- Strong SEO pillar structure

### Negative

- Limited to 3 segments at launch
- May need sub-segmentation later

### Risks

- Users who don't identify with a stage may bounce

## Compliance Checklist

- [ ] Aligns with privacy/security specs
- [ ] Aligns with performance budgets
- [ ] No new dependencies without justification

## Related

- [docs/specs/navigation-ia.md](../specs/navigation-ia.md)
- [docs/specs/page-templates.md](../specs/page-templates.md)

---

## Owner / Reviewers / Last updated

Owner: TBD | Reviewers: TBD | Last updated: 2026-02-20
