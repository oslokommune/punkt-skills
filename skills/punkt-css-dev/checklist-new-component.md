# Checklist: Adding Component Styles

1. Read the component spec in `component-specs/{name}.json` for required classes and modifiers.
2. Decide placement: `src/scss/components/` for UI patterns, `src/scss/elements/` for base HTML controls.
3. Create the partial: `_component-name.scss` (underscore prefix, kebab-case).
4. Follow the [component pattern template](component-patterns.md):
   - Namespaced imports (`@use 'sass:map'`, `@use '../abstracts/variables'`, etc.)
   - Private variables for cached spacing values
   - Custom element display rule (if web component exists)
   - BEM class structure (`.pkt-{name}`, `__element`, `--modifier`)
5. Use **semantic CSS custom properties** for colors (`var(--pkt-color-*)`).
6. Use the **`bp()` mixin** for responsive styles.
7. Use **`typography.get-text()`** for text styles.
8. Implement **dark mode** overrides where needed (see [Dark Mode](dark-mode.md)).
9. Implement **interactive states** with both pseudo-classes and BEM modifier classes (see [States](states.md)).
10. Add focus indicators using the double box-shadow pattern.
11. Register the partial in the directory's `_index.scss` via `@forward`.
12. Run `npm run build` from `packages/css/` to verify compilation.
