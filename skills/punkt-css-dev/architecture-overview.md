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
