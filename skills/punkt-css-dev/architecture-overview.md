# Architecture Overview

`@oslokommune/punkt-css` is the SCSS-based styling layer for Punkt, Oslo kommune's design system. It provides BEM-structured component styles, utility classes, a token system (colors, spacing, typography), and full dark mode support via CSS custom properties.

## Key facts

- **Language**: SCSS (Dart Sass, modern module system with `@use`/`@forward`)
- **No bundler**: Compiled directly with Dart Sass via a Node build script
- **No JavaScript**: Pure CSS output, no JS dependencies
- **BEM naming**: All classes prefixed with `pkt-`
- **Theming**: CSS custom properties for runtime theming and dark mode
- **Web Component ready**: Includes display rules for all custom elements (`pkt-button`, `pkt-card`, etc.)

## Relationship to other packages

| Package | Relationship |
|---|---|
| `@oslokommune/punkt-react` | Consumes BEM classes — React components apply `pkt-*` classes, never inline styles |
| `@oslokommune/punkt-elements` | Lit web components consume the same BEM classes |
| `@oslokommune/punkt-assets` | Provides fonts (Oslo Sans) and icons referenced by CSS |
| `component-specs/` | JSON specs define which CSS classes a component needs |

## Module system

The package uses the Sass module system exclusively:
- `@use` for importing dependencies (replaces `@import`)
- `@forward` for re-exporting from index files
- Namespaced access to variables (e.g., `variables.$spacing`)
- No global namespace pollution

### Do not `@use` a partial from another folder

`@use 'abstracts/…'` is always fine — abstracts emit no CSS. But `@use`-ing a **partial that
emits CSS** from another folder duplicates that CSS into every bundle the importer belongs to.

`components/_textinput.scss` did `@use '../elements/input'`. `_input.scss` lives in `elements/`
and is forwarded from `elements/_index.scss`, so `pkt-elements.css` had it once — but the `@use`
also pulled **72 `.pkt-input` rules into `pkt-components.css`**, where they had no business
being. Anyone loading both bundles got the rules twice, with the stray copy winning on source
order. Removing the file cut 11 956 bytes from `pkt-components`.

`pkt.css` never showed the problem: it loads `elements/input` as a module once, so the second
`@use` resolved to the already-loaded module and emitted nothing. The duplication only appears
in the per-bundle files — which is exactly where nobody looks.

If a component needs styles from another folder, apply that component's class in the markup, or
move the shared declarations into a mixin under `abstracts/`. There are currently **no
cross-folder `@use`s** in `components/` or `elements/`; keep it that way.
