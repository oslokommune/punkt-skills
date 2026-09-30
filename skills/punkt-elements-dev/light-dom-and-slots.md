# Light DOM & Slot Content Distribution

## Light DOM rendering

Most Punkt elements render in **light DOM** by extending `PktElement`, which overrides `createRenderRoot()` to return `this`. This means:

- Component output is rendered directly into the host element (no shadow root)
- Global CSS classes from `@oslokommune/punkt-css` apply naturally
- Native Shadow DOM `<slot>` elements are **not available**
- Content distribution is handled by the `slotContent` directive

## slotContent directive

**File:** `src/directives/slot-content.ts`

A Lit `AsyncDirective` that provides declarative slot-like content distribution without Shadow DOM. It collects the host element's children before Lit's first render and distributes them into designated positions in the template.

### How it works

1. Components that use slots extend `PktElementWithSlot` (instead of `PktElement`)
2. `PktElementWithSlot.connectedCallback()` calls `SlotManager.collectNodes()` to capture children before Lit renders
3. The component's template uses `${slotContent(this)}` to place default slot content
4. Named slots use `${slotContent(this, 'slotName')}`
5. A `MutationObserver` watches the host's subtree. New content counts when it is added as a direct child of the host; removal counts wherever the node sits (including after it has been distributed into the template)
6. Every slot change calls `requestUpdate()` on the host, so anything computed from slot content in `render()` (like `hasSlotContent()`) stays reactive in both directions: none → some and some → none
7. The directive uses a generation counter to avoid unnecessary DOM updates

### Architecture

- **`SlotManager`** — Per-host singleton (stored in a `WeakMap`). Collects and categorizes children by `slot` attribute, observes mutations, notifies registered directives, and notifies other managers that receive its slots via `forwardSlots()`.
- **`SlotContentDirective`** — `AsyncDirective` that renders collected nodes at its position. Returns `noChange` when content hasn't changed.
- **`ForwardSlotsDirective`** (`forwardSlots()`) — Element directive that hands a host's slot content to a child element's SlotManager without any intermediate element.
- **`getSlotManager(host)`** — Returns or creates the SlotManager for a host element.

**Known race:** the observer is started in a `setTimeout(0)` after the first render, because Lit's own light DOM render would otherwise be mistaken for user content. Content appended in the same tick as the first render is not picked up. Tests that add slot content dynamically must wait a macrotask first.

### Basic usage (single default slot)

```typescript
import { PktElementWithSlot } from '@/base-elements/element-with-slot'
import { slotContent } from '@/directives/slot-content'
import { html } from 'lit'
import { customElement } from 'lit/decorators.js'

export class PktExample extends PktElementWithSlot<IPktExample> {
  render() {
    return html`
      <div class="pkt-example">
        <span>${slotContent(this)}</span>
      </div>
    `
  }
}

try {
  customElement('pkt-example')(PktExample)
} catch (e) {
  console.warn('Forsøker å definere <pkt-example>, men den er allerede definert')
}
```

Consumer HTML:

```html
<pkt-example>Hello world</pkt-example>
<!-- "Hello world" is placed into the <span> -->
```

### Multiple named slots

```typescript
export class PktCard extends PktElementWithSlot<IPktCard> {
  render() {
    return html`
      <div class="pkt-card">
        <div class="pkt-card__header">${slotContent(this, 'header')}</div>
        <div class="pkt-card__body">${slotContent(this)}</div>
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

### Checking if a slot has content

Use the `hasSlotContent()` method on `PktElementWithSlot` to conditionally render based on whether a slot has content:

```typescript
render() {
  const classes = classMap({
    'pkt-inputwrapper__has-helptext':
      this.helptext || this.helptextDropdown || this.hasSlotContent(),
  })
  // ...
}
```

`hasSlotContent()` includes content forwarded to the element with `forwardSlots()`, and it is reactive: the host re-renders when content is added or removed.

### Forwarding slots to a child element (`forwardSlots`)

To pass a host's slot content on to a child component, use the `forwardSlots()` element directive on the child. **Do not render a holder element** like `<div slot="helptext">${slotContent(this, 'helptext')}</div>` inside the child: the holder keeps its `slot` attribute when the child forwards it further, so the next level files it under the wrong slot. That bug made all slotted helptext render outside `.pkt-inputwrapper__helptext` before `forwardSlots()` existed.

```typescript
import { forwardSlots } from '@/directives/slot-content'

// In textinput/textarea/select/combobox/datepicker/timepicker: same slot name in the child
render() {
  return html`
    <pkt-input-wrapper ${forwardSlots(this, ['helptext'])} ...>
      <!-- other content -->
    </pkt-input-wrapper>
  `
}
```

```typescript
// In input-wrapper: the host's helptext slot becomes pkt-helptext's default slot (null)
const helptextElement = () => {
  if (!hasHelptext && !this.helptextDropdown) return nothing
  return html`<pkt-helptext ${forwardSlots(this, { helptext: null })} ...></pkt-helptext>`
}
```

Rules:

- The child renders the forwarded nodes with its own `slotContent()`. The parent must not render the same slot with `slotContent()` as well.
- Forwarding chains: textinput → input-wrapper → pkt-helptext works, and changes propagate through every level.
- Rendering the child conditionally is safe. The forwarded content is handed over again when the child is rendered anew, and the parent keeps observing its own children while any child subscribes.
- A component that describes its control with the helptext must include slotted helptext: `hasHelptext: !!this.helptext || this.hasSlotContent('helptext')`.

## Slot content reactivity

Slot content wrapped in a container element (div/span) maintains Lit template bindings when moved by the directive. **Always wrap reactive slot content in a container element:**

```html
<!-- Good: wrapper div preserves Lit's template reference -->
<pkt-alert>
  <div>${dynamicContent}</div>
</pkt-alert>

<!-- Bad: bare text/expressions lose their binding when moved -->
<pkt-alert> ${dynamicContent} </pkt-alert>
```

Without a wrapper, Lit loses track of the template parts when nodes are moved, and subsequent re-renders may duplicate or fail to update content.

## Automatic filtering

The `slotContent` directive **always** filters out:

- `<option>` and `<data>` elements (handled by `PktOptionsSlotController`)
- Elements with the `data-skip` attribute
- Elements injected by the dialog polyfill (`_dialog_overlay`, `backdrop`)
- Empty/whitespace-only text nodes

This means components that accept both slot content and `<option>` children (like `pkt-select` and `pkt-combobox`) do not need any special configuration.

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
import { PktOptionsSlotController } from '@/controllers/pkt-options-controller'
import { forwardSlots } from '@/directives/slot-content'

export class PktSelect extends PktOptionsInputElement<{}, TSelectOption> {
  constructor() {
    super()
    this.optionsController = new PktOptionsSlotController(this)
  }

  connectedCallback(): void {
    super.connectedCallback()
    this.parseOptions()
  }

  render() {
    return html`
      <pkt-input-wrapper ${forwardSlots(this, ['helptext'])} ...>
        <!-- select UI -->
      </pkt-input-wrapper>
    `
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

### Option type

```typescript
type TOption = {
  value: string
  label: string
  selected?: boolean
  disabled?: boolean
  hidden?: boolean // preserved from data-hidden attribute
}
```

## Slot utility functions

**File:** `src/controllers/pkt-slot-utils.ts`

Helper functions used internally by the directive and options controller:

| Function                         | Purpose                                                |
| -------------------------------- | ------------------------------------------------------ |
| `shouldSkip(element)`            | Whether to skip element (dialog polyfill, `data-skip`) |
| `isOptionElement(element)`       | Is `<option>` or `<data>`                              |
| `isTextNodeAndNotEmpty(element)` | Is a non-empty text node                               |
