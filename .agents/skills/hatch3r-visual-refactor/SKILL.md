---
name: hatch3r-visual-refactor
description: UI/UX change workflow matching design, accessibility, and responsiveness requirements. Use when making visual changes, updating components, working on UI issues, or implementing design mockups.
---

# Visual Refactor Workflow

## Quick Start

```
Task Progress:
- [ ] Step 1: Read the issue, mockups, and design system
- [ ] Step 2: Produce a visual change plan
- [ ] Step 3: Implement matching the mockup
- [ ] Step 4: Verify accessibility and responsiveness
- [ ] Step 5: Open PR with before/after screenshots
```

## Step 1: Read Inputs

- Parse the issue body: proposed changes, before/after mockups, affected surfaces, accessibility checklist, responsiveness requirements.
- Read project quality documentation (accessibility, animation budgets).
- Review the existing design system tokens and component hierarchy.
- For external library docs and current best practices, follow the project's tooling hierarchy.

## Step 2: Visual Change Plan

Before modifying code, output:

- **Surfaces affected:** list with stable IDs
- **Components to modify/create:** list (prefer modifying existing)
- **Design tokens used:** colors, spacing, typography
- **Accessibility approach:** how WCAG AA compliance is achieved
- **Responsiveness:** how it adapts across widget/panel sizes
- **Animation changes:** new/modified animations, frame budget

## Step 3: Implement

- Match the mockup/screenshot exactly. Do not improvise design.
- Use existing design system tokens and components.
- Ensure animations respect `prefers-reduced-motion`.
- Ensure color contrast meets WCAG AA (4.5:1 for text).
- Ensure interactive elements are keyboard accessible with focus indicators.
- Add ARIA attributes for screen reader support.

## Step 4: Verify

- Snapshot tests updated for all modified components.
- No visual regressions in unaffected surfaces.
- Accessibility: contrast, ARIA, keyboard nav verified.
- Animations at 60fps (if applicable).

```bash
npm run lint && npm run typecheck && npm run test
```

## Step 5: Open PR

Use the project's PR template. Include:

- Before/after screenshots (required)
- Accessibility verification evidence
- Responsive behavior across sizes

## Definition of Done

- [ ] UI matches mockup/design in the issue
- [ ] Color contrast >= 4.5:1 (WCAG AA)
- [ ] Animations respect `prefers-reduced-motion`
- [ ] Interactive elements keyboard accessible
- [ ] ARIA attributes for screen readers
- [ ] Responsive across applicable host sizes
- [ ] Snapshot tests updated
- [ ] No visual regressions
- [ ] Design system tokens used (no ad-hoc styling)
