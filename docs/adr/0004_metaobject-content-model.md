# ADR-0004: Shopify Metaobjects for Education-as-Conversion Content

## Purpose

This ADR records the decision to use Shopify Metaobjects for reusable education content, enabling single-source-of-truth content that supports the store's conversion strategy.

---

- **Status:** ACCEPTED
- **Date:** 2026-02-20
- **Decision Makers:** TBD

## Context

The store's conversion strategy depends on education (ingredient glossary, clinical callouts, advisor profiles, FAQs). This content must be reusable across pages (PDP, glossary, stage guide) and maintainable by non-developers. Hardcoding in Liquid templates would create duplication and maintenance burden.

## Decision

Use Shopify Metaobjects for five content types: Ingredients, Clinical Callouts, Advisor Profiles, FAQs, and Flavor Variants. Reference these from sections and templates via metaobject lists and references.

## Alternatives Considered

| Option | Pros | Cons |
|--------|------|------|
| Hardcoded Liquid sections | Simple, no setup | Duplication, non-developer unfriendly, hard to maintain |
| Blog/article-based content | Built-in Shopify feature, SEO-friendly | Limited structured fields, poor reusability across pages |
| External CMS (Contentful, Sanity) | Powerful content modeling | Added complexity, cost, latency, extra integration |

## Consequences

### Positive

- Single source of truth
- Reusable across templates
- Merchant-editable

### Negative

- Metaobject UI can be clunky for large datasets

### Risks

- Shopify metaobject API limits for very large catalogs

## Compliance Checklist

- [ ] Aligns with privacy/security specs
- [ ] Aligns with performance budgets
- [ ] No new dependencies without justification

## Related

- [docs/specs/shopify-data-model.md](../specs/shopify-data-model.md)
- [docs/specs/page-templates.md](../specs/page-templates.md)

---

## Owner / Reviewers / Last updated

Owner: TBD | Reviewers: TBD | Last updated: 2026-02-20
