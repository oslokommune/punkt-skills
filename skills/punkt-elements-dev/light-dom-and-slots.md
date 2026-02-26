# Light DOM & Slot Controllers

## Light DOM rendering

Most Punkt elements render in **light DOM** by extending `PktElement`, which overrides `createRenderRoot()` to return `this`. This means:

- Component output is rendered directly into the host element (no shadow root)
- Global CSS classes from `@oslokommune/punkt-css` apply naturally
- Native Shadow DOM `<slot>` elements are **not available**
- Content distribution must be handled by `PktSlotController`

## PktSlotController

**File:** `src/controllers/pkt-slot-controller.ts`

A Lit reactive controller that provides slot-like content distribution without Shadow DOM. It watches the host's children and moves them into designated container elements based on their `slot` attribute.

### How it works

1. The component creates `Ref<HTMLElement>` objects for each "slot container"
2. `PktSlotController` is instantiated with references to these containers
3. On connect, the controller collects all child nodes of the host
4. It distributes nodes to matching containers based on `slot` attribute
5. A `MutationObserver` watches for child changes and re-distributes

### Basic usage (single default slot)

```typescript
import { PktElement } from '@/base-elements/element'
import { PktSlotController } from '@/controllers/pkt-slot-controller'
import { html } from 'lit'
import { customElement } from 'lit/decorators.js'
import { createRef, Ref, ref } from 'lit/directives/ref.js'

@customElement('pkt-example')
export class PktExample extends PktElement<IPktExample> {
  defaultSlot: Ref<HTMLElement> = createRef()
  slotController: PktSlotController

  constructor() {
    super()
    this.slotController = new PktSlotController(this, this.defaultSlot)
  }

  render() {
    return html`
      <div class="pkt-example">
        <span ${ref(this.defaultSlot)}></span>
      </div>
    `
  }
}
```

Consumer HTML:
```html
<pkt-example>Hello world</pkt-example>
<!-- "Hello world" is moved into the <span> -->
```

### Multiple named slots

```typescript
@customElement('pkt-card')
export class PktCard extends PktElement<IPktCard> {
  defaultSlot: Ref<HTMLElement> = createRef()
  headerSlot: Ref<HTMLElement> = createRef()
  slotController: PktSlotController

  constructor() {
    super()
    this.slotController = new PktSlotController(this, this.defaultSlot, this.headerSlot)
  }

  render() {
    return html`
      <div class="pkt-card">
        <div class="pkt-card__header" ${ref(this.headerSlot)} name="header"></div>
        <div class="pkt-card__body" ${ref(this.defaultSlot)}></div>
      </div>
    `
  }
}
```

Consumer HTML:
```html
<pkt-card>
  <h2 slot="header">Card Title</h2>
  <p>Card body content goes to default slot</p>
</pkt-card>
```

### Tracking filled slots

The controller maintains a `filledSlots` Set that tracks which slots currently have content. If the host implements an `updateSlots(filledSlots)` method, it will be called whenever slots change:

```typescript
updateSlots(filledSlots: Set<string | null | undefined>) {
  this.hasHeader = filledSlots.has('header')
  this.requestUpdate()
}
```

### skipOptions flag

For input components that accept both slotted content **and** `<option>` children, set `skipOptions = true` so the slot controller ignores `<option>` elements (they are handled by `PktOptionsSlotController` instead):

```typescript
constructor() {
  super()
  this.optionsController = new PktOptionsSlotController(this)
  this.slotController = new PktSlotController(this, this.helptextSlot)
  this.slotController.skipOptions = true
}
```

## Slot content reactivity

Slot content is distributed once and is **not reactive** to Lit property changes by default. If slot content needs to update when the component's state changes, it must be wrapped in a container element so Lit can re-render the content in place. The recommended pattern uses a `<div class="pkt-contents">` wrapper:

```html
<pkt-card>
  <div class="pkt-contents">
    <h2>{responsiveTitle}</h2>
    <p>{responsiveContent}</p>
    {metaSection}
  </div>
</pkt-card>
```

Without a wrapper, individual text nodes or elements passed as slot children are moved by the slot controller and lose their binding to the parent component's render cycle.

## PktOptionsSlotController

**File:** `src/controllers/pkt-options-controller.ts`

A reactive controller for components that accept `<option>` or `<data>` elements as children. It extracts these elements, hides them from the DOM, and converts them into a typed option array.

### How it works

1. On connect, collects all `<option>` and `<data>` child elements
2. Hides each element (adds `hidden`, `data-skip`, `pkt-hide` class)
3. Converts them to `{ value, label, selected, disabled, hidden }` objects
4. A `MutationObserver` watches for added/removed options
5. Calls `host.requestUpdate()` when options change

### Usage

```typescript
import { PktOptionsInputElement } from '@/base-elements/options-input-element'
import { PktOptionsSlotController } from '@/controllers/pkt-options-controller'
import { PktSlotController } from '@/controllers/pkt-slot-controller'

@customElement('pkt-select')
export class PktSelect extends PktOptionsInputElement<{}, TSelectOption> {
  private helptextSlot: Ref<HTMLElement> = createRef()

  constructor() {
    super()
    this.optionsController = new PktOptionsSlotController(this)
    this.slotController = new PktSlotController(this, this.helptextSlot)
    this.slotController.skipOptions = true
  }

  connectedCallback(): void {
    super.connectedCallback()
    this.parseOptions()  // Inherited from PktOptionsInputElement
  }
}
```

Consumer HTML:
```html
<pkt-select label="Choose country">
  <option value="no" selected>Norge</option>
  <option value="se">Sverige</option>
  <option value="dk">Danmark</option>
</pkt-select>
```

The `<option>` elements are hidden and converted to:
```json
[
  { "value": "no", "label": "Norge", "selected": true },
  { "value": "se", "label": "Sverige" },
  { "value": "dk", "label": "Danmark" }
]
```

### Option type

```typescript
type TOption = {
  value: string
  label: string
  selected?: boolean
  disabled?: boolean
  hidden?: boolean  // preserved from data-hidden attribute
}
```

## Slot utility functions

**File:** `src/controllers/pkt-slot-utils.ts`

Helper functions used internally by the controllers:

| Function | Purpose |
|---|---|
| `shouldSkip(element)` | Whether to skip element (dialog polyfill, `data-skip`) |
| `isOptionElement(element)` | Is `<option>` or `<data>` |
| `isSlotElement(element, slots)` | Is the element a slot container |
| `isNodeAndNotSlot(element, slots)` | Is a distributable content node |
| `isTextNodeAndNotEmpty(element)` | Is a non-empty text node |
