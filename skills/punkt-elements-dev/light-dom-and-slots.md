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
5. A `MutationObserver` watches for dynamic child changes and re-distributes
6. The directive uses a generation counter to avoid unnecessary DOM updates

### Architecture

- **`SlotManager`** — Per-host singleton (stored in a `WeakMap`). Collects and categorizes children by `slot` attribute, observes mutations, notifies registered directives.
- **`SlotContentDirective`** — `AsyncDirective` that renders collected nodes at its position. Returns `noChange` when content hasn't changed.
- **`getSlotManager(host)`** — Returns or creates the SlotManager for a host element.

### Basic usage (single default slot)

```typescript
import { PktElementWithSlot } from '@/base-elements/element'
import { slotContent } from '@/directives/slot-content'
import { html } from 'lit'
import { customElement } from 'lit/decorators.js'

@customElement('pkt-example')
export class PktExample extends PktElementWithSlot<IPktExample> {
  render() {
    return html`
      <div class="pkt-example">
        <span>${slotContent(this)}</span>
      </div>
    `
  }
}
```

Consumer HTML:
```html
<pkt-example>Hello world</pkt-example>
<!-- "Hello world" is placed into the <span> -->
```

### Multiple named slots

```typescript
@customElement('pkt-card')
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

### Forwarding helptext slots through component layers

Several input components (textinput, textarea, datepicker, select, combobox) forward a `helptext` named slot through to `pkt-input-wrapper`, which in turn forwards it to `pkt-helptext`. The pattern is:

```typescript
// In textinput/textarea/etc:
render() {
  return html`
    <pkt-input-wrapper ...>
      <div class="pkt-contents" slot="helptext">${slotContent(this, 'helptext')}</div>
      <!-- other content -->
    </pkt-input-wrapper>
  `
}
```

```typescript
// In input-wrapper:
const helptextElement = () => {
  return html`
    <pkt-helptext ...>${slotContent(this, 'helptext')}</pkt-helptext>
  `
}
```

## Slot content reactivity

Slot content wrapped in a container element (div/span) maintains Lit template bindings when moved by the directive. **Always wrap reactive slot content in a container element:**

```html
<!-- Good: wrapper div preserves Lit's template reference -->
<pkt-alert>
  <div>${dynamicContent}</div>
</pkt-alert>

<!-- Bad: bare text/expressions lose their binding when moved -->
<pkt-alert>
  ${dynamicContent}
</pkt-alert>
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
import { slotContent } from '@/directives/slot-content'

@customElement('pkt-select')
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
      <pkt-input-wrapper ...>
        <div class="pkt-contents" slot="helptext">${slotContent(this, 'helptext')}</div>
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
  hidden?: boolean  // preserved from data-hidden attribute
}
```

## Slot utility functions

**File:** `src/controllers/pkt-slot-utils.ts`

Helper functions used internally by the directive and options controller:

| Function | Purpose |
|---|---|
| `shouldSkip(element)` | Whether to skip element (dialog polyfill, `data-skip`) |
| `isOptionElement(element)` | Is `<option>` or `<data>` |
| `isTextNodeAndNotEmpty(element)` | Is a non-empty text node |
