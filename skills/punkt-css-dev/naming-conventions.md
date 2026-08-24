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

## Always select by class, never by custom element name

Style `.pkt-timepicker`, never `pkt-timepicker`. Same for attributes: use a modifier class,
not `[fullwidth]`.

Reason: only Elements renders custom elements and their attributes. React renders plain
`div`s with classes, so an element- or attribute-based selector silently does nothing there —
and the two packages are meant to produce identical markup apart from the wrapper tag.

Sole exception: `display` on the host element, since custom elements are inline by default and
React has no equivalent element to fix. This exception covers attribute selectors too when the
value must be right before JS runs — `pkt-textinput[inline] { display: inline-block }` — but it
covers `display` only, never width or other layout properties.

## Form components: two boxes, component class on the outer one

Every form component has the same shape in both frameworks:

```
komponentboks (.pkt-textinput / .pkt-datepicker / .pkt-combobox-component …)
  └─ .pkt-inputwrapper
       └─ .pkt-inputwrapper__fieldset
```

In Elements the component class goes on the **host**, set with `classList.add()` in `updated()`;
in React it goes on an outer `div`. `pkt-input-wrapper` is itself the `.pkt-inputwrapper` box —
it renders no wrapper div of its own.

Width belongs on the outer component box, never on `.pkt-inputwrapper`: the outer box is the
flex/grid item, and a percentage width further in contributes nothing to shrink-to-fit sizing, so
the field collapses to its intrinsic width. Key such rules on the component class
(`.pkt-textarea:has(textarea…)`), which lands on the right box in both frameworks.

Host element selectors carry `display` only, as a pre-JS fallback before the class is set.

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
