---
id: hatch3r-component-conventions
type: rule
description: Component conventions for accessibility, design tokens, and responsive patterns
scope: always
---

# Component Conventions

## Structure

- Use framework-recommended component syntax (e.g., `<script setup>` for Vue).
- Define props with typed interfaces.
- Define emits/events with typed interfaces.
- Use composables/hooks from project composables directory — never mixins.
- Use stores or equivalent for shared state.

## Naming

- Component files: PascalCase (e.g., `PetWidget.vue`, `QuestPanel.vue`).
- Component names match file names.
- Props: camelCase. Events: kebab-case.

## Styling

- Use the project's design tokens for colors, spacing, typography.
- Prefer utility classes or scoped CSS with BEM naming.
- No inline styles except for dynamic values that can't be expressed as classes.
- No hardcoded color values — use tokens.

## Accessibility (Required)

- All animations: wrap in `prefers-reduced-motion` media query AND check user's `reducedMotion` setting.
- Color contrast: ≥ 4.5:1 for text (WCAG AA).
- Interactive elements: keyboard focusable with visible focus indicator.
- Dynamic content changes: use `aria-live` regions.
- Support high contrast mode.

## Performance

- UI must render at 60fps (≤ 16ms per frame).
- Prefer CSS animations/transitions over JavaScript animations.
- Use conditional visibility (e.g., `v-show`) over conditional mount (e.g., `v-if`) for frequently toggled elements.
- Lazy-load panels that aren't immediately visible.

## Testing

- Snapshot tests for all visual states.
- Component tests for interactive behavior.
- Verify reduced-motion behavior in tests.
