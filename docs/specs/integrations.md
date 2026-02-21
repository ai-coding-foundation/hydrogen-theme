# Integrations Specification

## Purpose

This spec defines the core third-party integrations for the EVOHERA Shopify store. The principle is **one integration per category**—keep it lean and avoid bloat. The stack matches the subscription + retention-heavy model.

---

## Integration Stack

| Category | Tool | Purpose | Key Integration Points |
|----------|------|---------|-------------------------|
| **Email/SMS** | Klaviyo | Lifecycle flows, subscription reminders, stage-based segmentation | Welcome series, subscription reminder flows, stage-based content, abandoned cart |
| **Subscriptions** | Recharge | Quarterly/monthly subscription management | Subscription selector on PDP, Manage Subscription portal, recurring billing |
| **Support** | Gorgias | Helpdesk, subscription management portal | Help Center, Manage Subscription, contact flows, ticket routing |
| **Reviews** | Okendo | Product reviews, UGC, social proof on PDP | PDP reviews block, UGC display, review request flows |

---

## Principles

- **One per category**: No duplicate tools (e.g., one email platform, one subscription platform).
- **Subscription + retention focus**: Stack supports recurring revenue, lifecycle communication, and support.
- **Lean**: Avoid adding integrations until there is a clear need.

---

## Acceptance Criteria

- [ ] Klaviyo configured for email/SMS; lifecycle and stage-based flows defined
- [ ] Recharge integrated; subscription selector on PDP with quarterly default
- [ ] Gorgias configured for support; Manage Subscription portal accessible
- [ ] Okendo integrated; reviews displayed on PDP
- [ ] No duplicate tools within any category

---

## Owner / Reviewers / Last updated

Owner: TBD | Reviewers: TBD | Last updated: 2026-02-20
