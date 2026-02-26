# Naming Conventions

## File naming

- **Partials**: `_filename.scss` (underscore prefix — not compiled directly)
- **Entry points**: `pkt.scss`, `pkt-base.scss` (no underscore — compiled to CSS)
- **Index files**: `_index.scss` (forwards all partials in a directory)
- **Components/elements**: `_accordion.scss`, `_button.scss` (kebab-case, singular)

## CSS class naming (BEM)

All classes use the `pkt-` prefix and strict BEM:

| Level | Pattern | Example |
|---|---|---|
| Block | `.pkt-{component}` | `.pkt-btn`, `.pkt-card` |
| Element | `.pkt-{component}__{element}` | `.pkt-btn__icon`, `.pkt-card__title` |
| Modifier | `.pkt-{component}--{modifier}` | `.pkt-btn--primary`, `.pkt-card--blue` |
| State | `.pkt-{component}--{state}` | `.pkt-btn--hover`, `.pkt-btn--disabled` |
| Utility | `.pkt-{utility}` | `.pkt-sr-only`, `.pkt-hide`, `.pkt-truncate-text` |

## Sass variable naming

| Scope | Pattern | Example |
|---|---|---|
| Public variable | `$kebab-case` | `$breakpoints`, `$font-family`, `$spacing` |
| Private variable | `$-kebab-case` (leading dash) | `$-module-name`, `$-spacing-8` |
| Map keys | Quoted strings | `'size-16'`, `'pkt-txt-18-medium'` |

## CSS custom property naming

Pattern: `--pkt-color-{category}-{name}[-{variant}]`

| Category | Example |
|---|---|
| `brand` | `--pkt-color-brand-blue-1000` |
| `text` | `--pkt-color-text-body-default` |
| `border` | `--pkt-color-border-default` |
| `surface` | `--pkt-color-surface-default-light-blue` |
| `background` | `--pkt-color-background-card` |
| `input` | `--pkt-color-input-border-hover` |
| `button` | `--pkt-color-button-background-normal` |

## Mixin / function naming

| Scope | Pattern | Example |
|---|---|---|
| Public mixin | `@mixin kebab-case` | `bp`, `get-text`, `truncate-text` |
| Private mixin | `@mixin -kebab-case` | `-print-skin-as-css-vars`, `-size` |
| Function | `kebab-case` | `map-deep-get`, `str-replace`, `escape-svg` |
