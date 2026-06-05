---
name: punkt-css-dev
description: "Developing component and utility styles in the Punkt design system (@oslokommune/punkt-css). Covers writing BEM component classes, creating SCSS mixins, defining design tokens, managing dark mode overrides, and maintaining responsive breakpoints in /packages/css/src/scss/. Use when working on Punkt CSS source code, adding new component styles, or modifying SCSS variables and partials."
---

# Punkt CSS Development

Skill for developing component and utility styles in the Punkt design system (`@oslokommune/punkt-css`). SCSS-based with BEM naming (`pkt-` prefix), Dart Sass module system (`@use`/`@forward`), CSS custom properties for theming and dark mode, no JavaScript. Components live in `src/scss/components/`, elements in `src/scss/elements/`.

## Quick Start: Component SCSS Pattern

```scss
@use 'sass:map';
@use '../abstracts/variables';
@use '../abstracts/mixins/breakpoints' as *;
@use '../abstracts/mixins/typography';

$-spacing-16: map.get(variables.$spacing, 'size-16');

pkt-component {
  display: block;
}

.pkt-component {
  padding: $-spacing-16;
  color: var(--pkt-color-text-body-default);

  &__title {
    @include typography.get-text('pkt-txt-18-medium');
  }

  &--compact {
    padding: map.get(variables.$spacing, 'size-8');
  }

  @include bp('tablet-up') {
    padding: map.get(variables.$spacing, 'size-24');
  }

  [data-mode='dark'] & {
    color: var(--pkt-color-brand-neutrals-white);
  }
}
```

## Workflow: Adding Component Styles

1. Read the component spec in `component-specs/{name}.json` for required classes
2. Create `src/scss/components/_component-name.scss` (or `elements/` for base controls)
3. Follow the pattern above — namespaced imports, private `$-` variables, BEM structure
4. Use semantic color tokens (`var(--pkt-color-*)`) and `bp()` mixin for responsive
5. Add dark mode overrides with `[data-mode='dark'] &` where needed
6. Register in the directory's `_index.scss` via `@forward`
7. Run `npm run build` from `packages/css/` to verify compilation

## Sections

1. [Architecture Overview](architecture-overview.md)
2. [Directory Structure](directory-structure.md)
3. [Naming Conventions](naming-conventions.md)
4. [Abstracts (Variables, Mixins, Functions)](abstracts.md)
5. [Component / Element SCSS Patterns](component-patterns.md)
6. [Colors & Theming](colors-and-theming.md)
7. [Dark Mode](dark-mode.md)
8. [Typography](typography.md)
9. [Spacing](spacing.md)
10. [Breakpoints & Responsive](breakpoints.md)
11. [Grid & Layout](grid-and-layout.md)
12. [States & Interactivity](states.md)
13. [Accessibility](accessibility.md)
14. [Build System](build-system.md)
15. [Checklist: Adding Component Styles](checklist-new-component.md)
16. [Astro Dev / Test Pages](astro-dev-pages.md)
