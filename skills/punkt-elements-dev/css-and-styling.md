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

## The host is a thin wrapper

`<pkt-card>` wraps the real root; it is not the root. Render the same markup React renders, and
leave the host carrying nothing:

```typescript
render() {
  return html`<div class="pkt-progressbar" role=${this.resolvedRole} aria-valuenow=${this.valueCurrent}>
    …
  </div>`
}
```

Rules that follow from this:

- **No classes, `role`, `aria-*` or `id` on the host.** A class there is invisible to React, and a
  duplicated `role` or `id` is an accessibility defect — two nested elements claiming the same role.
- **Never `setAttribute` onto `this`** to expose semantics. Render them on the inner root instead.
- **Relocating an author-set attribute?** Capture the value *before* `removeAttribute` —
  removal fires `attributeChangedCallback` and nulls the Lit property. See
  `PktButton.relocatedAriaAttributes` and `PktProgressbar.willUpdate` for the pattern.
- **Compute derived values in `render()`**, not into `@state` from `updated()`. State written after
  render is missing from the first paint.
- **The host needs a `display` rule** in `packages/css` — custom elements are `display: inline` by
  default. `display` only; never width or other layout properties.

`packages/conformance` enforces this: it renders React and Elements and fails on any
difference in the markup inside the host. Run `npm run test-conformance`.

**Not yet migrated:** the form components still put the component class on the host (see
[Input wrapper](#input-wrapper) below). They are logged as known divergences in the conformance suite.

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

Form input components use the `pkt-input-wrapper` component for consistent label, helptext, and error display.

**`pkt-input-wrapper` is itself the `.pkt-inputwrapper` box** — it renders no wrapper div of its
own, and its classes sit on the host. Every form component therefore has the same two boxes in
both frameworks: an outer component box carrying the component class, then `.pkt-inputwrapper`.

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

Five things every form component owes the wrapper:

**1. Put the component class on the host**, not on the inner wrapper — React puts it on its outer
div, so this is what keeps the two frameworks aligned:

```typescript
protected updated(changedProperties: PropertyValues): void {
  super.updated(changedProperties)
  this.classList.add('pkt-textinput')
}
```

**2. Build aria ids from the same base as `forId`.** The wrapper derives `{forId}-helptext` and
`{forId}-error` from what you pass as `forId`. Passing `this.id + '-input'` as `forId` while
building `aria-errormessage` from `this.id` points the reference at an element that does not
exist, and nothing fails loudly — check the ids resolve.

**3. Describe the control, not the label.** `aria-describedby` belongs on the input itself; on a
`<label>` it contributes nothing to the accessible description:

```typescript
aria-describedby=${ifDefined(
  describedByIds({ id: this.id + '-input', hasHelptext: !!this.helptext, hasCounter: showCounter }),
)}
```

**4. Only emit `aria-errormessage` when there is an error**, and pair it with `aria-invalid`.

**5. Resolve tri-state props before forwarding.** `counter` and `hasFieldset` are
`boolean | null` on `PktInputElement`: null means "derive a sensible default", explicit `false`
always turns it off. Use the base helpers so the derivation is visible:

```typescript
const showCounter = this.effectiveCounter(this.counterMaxLength > 0)
...
?counter=${showCounter}
?hasFieldset=${this.effectiveHasFieldset(false)}
```

`components/input-wrapper/wrapper-parity.test.ts` checks that every wrapper prop is forwarded by
every component in both frameworks. Leaving one out is allowed, but it has to be declared in that
file's `OMITTED` list with a reason.

## Styling changes

When visual changes are needed, update the corresponding SCSS in `packages/css/src/scss/` — do **not** add inline styles or component-scoped CSS. See the [css-dev skill](../punkt-css-dev.md) for CSS development patterns.

## Forbidden

- **No `static styles = css\`...\``** — the standard Lit pattern for scoped styles is not used
- **No inline `style` attributes** — unless there is an explicit technical necessity
- **No CSS files in component directories** — styles live in the CSS package
