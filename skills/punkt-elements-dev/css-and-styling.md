# CSS & Styling

## Core principle

All styling in Punkt elements comes from **external BEM classes** provided by `@oslokommune/punkt-css`. Components never define their own styles — no `static styles`, no inline styles, no CSS-in-JS.

This is possible because most components render in **light DOM** (via `PktElement`), where global stylesheets apply naturally.

## classMap directive

Use Lit's `classMap()` directive for dynamic class application:

```typescript
import { classMap } from 'lit/directives/class-map.js'

render() {
  const classes = {
    'pkt-btn': true,
    [`pkt-btn--${this.size}`]: !!this.size,
    [`pkt-btn--${this.skin}`]: !!this.skin,
    [`pkt-btn--${this.variant}`]: !!this.variant,
    [`pkt-btn--${this.color}`]: !!this.color,
    'pkt-btn--full': !!this.fullWidth,
    'pkt-btn--disabled': !!this.disabled,
    'pkt-btn--isLoading': !!this.isLoading,
  }

  return html`<button class=${classMap(classes)}>...</button>`
}
```

The `classMap` directive only includes classes whose value is truthy. Use `!!` to coerce non-boolean values.

## BEM naming

CSS classes follow BEM methodology with the `pkt-` prefix:

| Type | Pattern | Example |
|---|---|---|
| Block | `.pkt-{block}` | `.pkt-btn` |
| Element | `.pkt-{block}__{element}` | `.pkt-btn__text` |
| Modifier | `.pkt-{block}--{modifier}` | `.pkt-btn--primary` |
| Size modifier | `.pkt-{block}--{size}` | `.pkt-btn--small` |
| State modifier | `.pkt-{block}--{state}` | `.pkt-btn--disabled` |

## Host element classes

For Shadow DOM components (extending `PktShadowElement`), apply classes directly to the host element:

```typescript
private updateHostClasses() {
  this.classList.remove('pkt-heading', 'pkt-heading--small', 'pkt-heading--medium')
  this.classList.add('pkt-heading')
  if (this.size) this.classList.add(`pkt-heading--${this.size}`)
}
```

## Class utility for host elements

**File:** `src/utils/classutils.ts`

```typescript
import { updateClassAttribute } from '@/utils/classutils'

// Add/remove a class based on a condition
updateClassAttribute(this, 'pkt-hide', !this.visible)
```

Used for toggling visibility classes (`pkt-hide`) on the host element, important for screen reader behavior.

## Input wrapper

Form input components use the `pkt-input-wrapper` component for consistent label, helptext, and error display:

```typescript
import '@/components/input-wrapper'

render() {
  return html`
    <pkt-input-wrapper
      ?disabled=${this.disabled}
      ?hasError=${this.hasError}
      label=${ifDefined(this.label)}
      errorMessage=${ifDefined(this.errorMessage)}
      helptext=${ifDefined(this.helptext)}
      forId=${this.id + '-input'}
    >
      <input class="pkt-input" id=${this.id + '-input'} ... />
    </pkt-input-wrapper>
  `
}
```

## Styling changes

When visual changes are needed, update the corresponding SCSS in `packages/css/src/scss/` — do **not** add inline styles or component-scoped CSS. See the [css-dev skill](../punkt-css-dev.md) for CSS development patterns.

## Forbidden

- **No `static styles = css\`...\``** — the standard Lit pattern for scoped styles is not used
- **No inline `style` attributes** — unless there is an explicit technical necessity
- **No CSS files in component directories** — styles live in the CSS package
