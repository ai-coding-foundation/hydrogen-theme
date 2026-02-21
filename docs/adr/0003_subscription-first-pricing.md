# ADR-0003: Quarterly Subscription as Default Pricing Model

## Purpose

This ADR records the decision to default the product purchase selector to quarterly subscription, aligning pricing with supplement consumption behavior and maximizing LTV.

---

- **Status:** ACCEPTED
- **Date:** 2026-02-20
- **Decision Makers:** TBD

## Context

Supplement businesses depend on recurring revenue and high LTV. One-time purchases have low retention. The 90-day supplement behavior cycle means quarterly replenishment matches natural consumption. Biologica demonstrates subscription-as-product-feature (not checkout add-on) converts well.

## Decision

Default the PDP purchase selector to quarterly subscription with visible savings callout. Monthly subscription and one-time purchase remain available as secondary options.

## Alternatives Considered

| Option | Pros | Cons |
|--------|------|------|
| One-time default + subscription upsell | Lower barrier to first purchase | Low subscription conversion, lower LTV |
| Monthly default | Familiar subscription model | "Too much product" churn, doesn't match 90-day behavior |
| Membership model (flat fee + discounts) | Predictable revenue | Complex to implement, unfamiliar for supplements |

## Consequences

### Positive

- Higher LTV
- Matches replenishment cadence
- Reduces "too much product" churn

### Negative

- Higher perceived commitment may reduce trial

### Risks

- Payment friction for first-time buyers

## Compliance Checklist

- [ ] Aligns with privacy/security specs
- [ ] Aligns with performance budgets
- [ ] No new dependencies without justification

## Related

- [docs/specs/product-line.md](../specs/product-line.md)
- [docs/specs/integrations.md](../specs/integrations.md) (Recharge)

---

## Owner / Reviewers / Last updated

Owner: TBD | Reviewers: TBD | Last updated: 2026-02-20
