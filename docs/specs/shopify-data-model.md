# Shopify Data Model Specification

## Purpose

This spec defines the Shopify metaobject definitions and relationships that power the EVOHERA "education as conversion" system. Metaobjects enable dynamic, non-hardcoded content for ingredients, clinical callouts, advisors, FAQs, and flavor variants.

---

## Overview

Use **Shopify Metaobjects** to store education and product-related content. This enables:

- Ingredient glossary driven by metaobjects
- Product pages that pull clinical callouts, FAQs, and flavor data dynamically
- Advisor profiles and quotes reusable across templates
- No hardcoded content for education assets

---

## Metaobject Definitions

### Ingredients

| Field                  | Type                   | Description                            |
| ---------------------- | ---------------------- | -------------------------------------- |
| `name`                 | single_line_text       | Ingredient name                        |
| `description`          | multi_line_text        | What it is                             |
| `benefits`             | multi_line_text        | Why it's included                      |
| `evidence_notes`       | multi_line_text        | Research/evidence summary              |
| `included_in_formulas` | list.product_reference | Which products include this ingredient |
| `image`                | file_reference         | Optional ingredient image              |

### Clinical Callouts

| Field        | Type             | Description                                   |
| ------------ | ---------------- | --------------------------------------------- |
| `metric`     | single_line_text | The metric or claim (e.g., "500mg Vitamin C") |
| `context`    | multi_line_text  | Supporting context                            |
| `disclaimer` | single_line_text | Required disclaimer text                      |

### Advisor Profiles

| Field      | Type             | Description                |
| ---------- | ---------------- | -------------------------- |
| `name`     | single_line_text | Advisor full name          |
| `title`    | single_line_text | Credential/title           |
| `headshot` | file_reference   | Portrait image             |
| `quote`    | multi_line_text  | Quote for use in templates |

### FAQs

| Field      | Type             | Description                                            |
| ---------- | ---------------- | ------------------------------------------------------ |
| `question` | single_line_text | FAQ question                                           |
| `answer`   | multi_line_text  | FAQ answer                                             |
| `scope`    | single_line_text | "global" or product handle (for product-specific FAQs) |

### Flavor Variants

| Field             | Type             | Description                  |
| ----------------- | ---------------- | ---------------------------- |
| `name`            | single_line_text | Flavor name                  |
| `tasting_notes`   | multi_line_text  | Flavor description           |
| `pairing`         | single_line_text | Suggested pairing or context |
| `acidity_scale`   | number_integer   | 1–5 scale                    |
| `sweetness_scale` | number_integer   | 1–5 scale                    |

---

## Relationships

| Relationship                     | Description                                                                                   |
| -------------------------------- | --------------------------------------------------------------------------------------------- |
| **Ingredients → Products**       | `included_in_formulas` links ingredients to products; enables "filter by formula" in glossary |
| **Products → Ingredients**       | Products reference ingredients for "What's inside" tabbed content                             |
| **Products → Clinical Callouts** | Products can display relevant clinical callouts                                               |
| **Products → FAQs**              | FAQs with `scope` = product handle display on that PDP                                        |
| **Products → Flavor Variants**   | Flavor variants link to product variants for tasting notes and scales                         |
| **Templates → Metaobjects**      | Sections pull from metaobjects via Shopify Liquid or Storefront API                           |

---

## Acceptance Criteria

- [ ] Ingredients metaobject defined with all fields
- [ ] Clinical Callouts metaobject defined with all fields
- [ ] Advisor Profiles metaobject defined with all fields
- [ ] FAQs metaobject defined with question, answer, scope
- [ ] Flavor Variants metaobject defined with all fields (acidity/sweetness 1–5)
- [ ] Relationships documented: ingredients ↔ products, FAQs ↔ products, flavors ↔ variants
- [ ] Education content (glossary, PDP) pulls from metaobjects, not hardcoded

---

## Owner / Reviewers / Last updated

Owner: TBD | Reviewers: TBD | Last updated: 2026-02-20
