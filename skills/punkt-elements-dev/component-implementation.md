# Component Implementation

## Naming conventions

- **Class name**: `Pkt` prefix + PascalCase. Example: `PktButton`, `PktAccordionItem`.
- **Custom element tag**: `pkt-` prefix + kebab-case. Example: `pkt-button`, `pkt-accordion-item`.
- **Interface name**: `IPkt` prefix + ComponentName. Example: `IPktButton`.
- **Type alias name**: `T` prefix. Example: `TPktButtonSkin`, `TSelectOption`.
- **File name**: kebab-case matching the component. Example: `button.ts`, `accordion-item.ts`.
- **Directory name**: kebab-case. Example: `button/`, `accordion/`.
- **CSS class prefix**: `pkt-` + kebab-case (from CSS package). Example: `pkt-btn`, `pkt-alert`.

## Component template (display component)

```typescript
import { PktElement } from '@/base-elements/element'
import { PktSlotController } from '@/controllers/pkt-slot-controller'
import { html } from 'lit'
import { customElement, property } from 'lit/decorators.js'
import { classMap } from 'lit/directives/class-map.js'
import { createRef, Ref, ref } from 'lit/directives/ref.js'

export type TPktExampleSkin = 'primary' | 'secondary'

export interface IPktExample {
  skin?: TPktExampleSkin
  label?: string
}

@customElement('pkt-example')
export class PktExample extends PktElement<IPktExample> implements IPktExample {
  defaultSlot: Ref<HTMLElement> = createRef()
  slotController: PktSlotController

  @property({ type: String }) skin: TPktExampleSkin = 'primary'
  @property({ type: String }) label: string = ''

  constructor() {
    super()
    this.slotController = new PktSlotController(this, this.defaultSlot)
  }

  render() {
    const classes = {
      'pkt-example': true,
      [`pkt-example--${this.skin}`]: !!this.skin,
    }

    return html`
      <div class=${classMap(classes)}>
        <span class="pkt-example__label">${this.label}</span>
        <div class="pkt-example__content" ${ref(this.defaultSlot)}></div>
      </div>
    `
  }
}

export default PktExample
```

## Required patterns

1. **`@customElement('pkt-*')`** decorator on every component class.
2. **Extend the correct base class** — `PktElement` for display, `PktInputElement` for form inputs, `PktOptionsInputElement` for option-based inputs.
3. **`implements IPkt*`** — implement the component's public interface for type safety.
4. **Generic type parameter** — pass the interface as `<IPktExample>` to the base class.
5. **`PktSlotController`** in constructor for any component accepting children.
6. **`classMap()`** directive for dynamic CSS classes.
7. **`ref()`** directive for DOM references (slot containers, input refs).
8. **Default export** at the bottom of the file.

## Decorators

### `@customElement('pkt-*')`
Registers the class as a custom element. Used on every component.

### `@property()`
Reactive property that can be set via HTML attribute or JavaScript.

```typescript
// Basic types
@property({ type: String }) label: string = ''
@property({ type: Number }) count: number = 0
@property({ type: Boolean, reflect: true }) disabled: boolean = false

// Reflect to attribute (syncs property → attribute)
@property({ type: String, reflect: true }) state: string = 'normal'

// Custom attribute name
@property({ type: Boolean, attribute: 'full-width' }) fullWidth: boolean = false
@property({ type: String, attribute: 'tag-skin' }) tagSkin: string = 'blue'

// Custom converter
@property({
  type: String,
  converter: {
    fromAttribute: (value: string | null): TSkin => value as TSkin,
    toAttribute: (value: TSkin): string => value,
  },
}) skin: TSkin = 'default'

// Array / Object props (not reflected to attributes)
@property({ type: Array }) options: TOption[] = []
@property({ type: Object }) image: { src: string; alt: string } = { src: '', alt: '' }
```

### `@state()`
Internal reactive state. Not exposed as an attribute. Triggers re-render on change.

```typescript
@state() private touched: boolean = false
@state() private tabCount: number = 0
```

### `@query()` / `@queryAll()`
Query elements from the rendered output. Rarely used — prefer `Ref` with `ref()` directive.

## Lifecycle methods

```typescript
// 1. Constructor — initialize controllers
constructor() {
  super()
  this.slotController = new PktSlotController(this, this.defaultSlot)
}

// 2. connectedCallback — setup listeners, parse initial state
connectedCallback(): void {
  super.connectedCallback()
  this.addEventListener('click', this.handleClick)
}

// 3. disconnectedCallback — cleanup
disconnectedCallback(): void {
  super.disconnectedCallback()
  this.removeEventListener('click', this.handleClick)
}

// 4. firstUpdated — after first render, DOM is available
protected firstUpdated(_changedProperties: PropertyValues): void {
  super.firstUpdated(_changedProperties)
  // Setup that needs rendered DOM
}

// 5. updated — after every render
protected updated(_changedProperties: PropertyValues): void {
  super.updated(_changedProperties)
  if (_changedProperties.has('skin')) {
    // React to property changes
  }
}

// 6. attributeChangedCallback — handle raw attribute changes
attributeChangedCallback(name: string, _old: string | null, value: string | null): void {
  super.attributeChangedCallback(name, _old, value)
  if (name === 'value' && this.value !== _old) {
    this.valueChanged(value, _old)
  }
}
```

**Always call `super.*()` in lifecycle methods** — the base classes have important logic in these methods.

## Render patterns

### Single render method (simple components)
```typescript
render() {
  const classes = { 'pkt-tag': true, [`pkt-tag--${this.skin}`]: !!this.skin }
  return html`<span class=${classMap(classes)} ${ref(this.defaultSlot)}></span>`
}
```

### Decomposed render helpers (complex components)
```typescript
render() {
  return html`
    <article class="pkt-card">
      ${this.renderImage()}
      ${this.renderHeader()}
      ${this.renderContent()}
    </article>
  `
}

private renderImage() {
  if (!this.image?.src) return nothing
  return html`<img class="pkt-card__image" src=${this.image.src} alt=${this.image.alt} />`
}
```

### Conditional rendering
```typescript
${this.isLoading ? html`<pkt-icon name="spinner"></pkt-icon>` : nothing}
${this.items.length > 0 ? html`<ul>...</ul>` : html`<p>No items</p>`}
```

Use `nothing` (from `lit`) instead of empty string when you want to render nothing.

## Forbidden patterns

- **No `static styles`** — all styling via BEM classes from `@oslokommune/punkt-css`.
- **No inline styles** — update the CSS package when styling changes are needed.
- **No `console.log`** or `alert()` — enforced by ESLint.
- **No bare `any`** — use `@typescript-eslint/no-explicit-any` escape comment when truly necessary.
