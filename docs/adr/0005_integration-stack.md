# ADR-0005: Klaviyo + Recharge + Gorgias + Okendo Integration Stack

## Purpose

This ADR records the decision to adopt a focused integration stack for email/SMS, subscriptions, support, and reviews, following the "one per category" principle.

---

- **Status:** ACCEPTED
- **Date:** 2026-02-20
- **Decision Makers:** TBD

## Context

The subscription-first, retention-heavy business model requires robust integrations for email/SMS, subscription management, customer support, and social proof. The "one per category" principle avoids integration bloat and conflicting scripts.

## Decision

Adopt four core integrations: Klaviyo (email/SMS), Recharge (subscriptions), Gorgias (support), Okendo (reviews).

## Alternatives Considered

| Category | Chosen | Alternative | Pros of Chosen | Cons of Alternative |
|----------|--------|-------------|----------------|---------------------|
| Email/SMS | Klaviyo | Mailchimp | Superior Shopify integration, subscription event triggers, stage-based segmentation | Mailchimp cheaper but weaker Shopify flows |
| Subscriptions | Recharge | Bold Subscriptions | Market leader for Shopify subscriptions, checkout integration | Bold has fewer features, smaller ecosystem |
| Support | Gorgias | Zendesk | Purpose-built for Shopify, native order/subscription sidebar | Zendesk more general, requires custom integration |
| Reviews | Okendo | Yotpo | Strong UGC + attributes for supplements | Yotpo more expensive at scale, broader feature set |

## Consequences

### Positive

- Proven Shopify ecosystem
- Mutual integrations between tools

### Negative

- Vendor lock-in
- Monthly SaaS cost

### Risks

- Recharge pricing changes
- Klaviyo deliverability

## Compliance Checklist

- [ ] Aligns with privacy/security specs
- [ ] Aligns with performance budgets
- [ ] No new dependencies without justification

## Related

- [docs/specs/integrations.md](../specs/integrations.md)
- [ADR-0003: Quarterly Subscription as Default Pricing Model](./0003_subscription-first-pricing.md)

---

## Owner / Reviewers / Last updated

Owner: TBD | Reviewers: TBD | Last updated: 2026-02-20
