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
5. A `MutationObserver` watches the host's subtree for as long as the host is connected. New content counts when it is added as a direct child of the host, or when other code inserts it inside a part that holds the host's content (see _Framework anchors_). Removal counts wherever the node sits, including after it has been distributed into the template
6. A reactive controller on the host separates Lit's own render from everybody else's changes: pending mutations are handled right before the host renders, and nodes Lit adds or removes while rendering are never taken for slot content changes
7. Every slot change calls `requestUpdate()` on the host, so anything computed from slot content in `render()` (like `hasSlotContent()`) stays reactive in both directions: none → some and some → none
8. The directive uses a generation counter to avoid unnecessary DOM updates

### Architecture

- **`SlotManager`** — Per-host singleton (stored in a `WeakMap`). Collects and categorizes children by `slot` attribute, observes mutations, notifies registered directives, and notifies other managers that receive its slots via `forwardSlots()`.
- **`SlotContentDirective`** — `AsyncDirective` that moves the collected nodes into its part itself and always returns `noChange`. It never hands Lit the node list: Lit would then clear everything between the part markers on each change, including nodes it did not put there (see _Framework anchors_ below).
- **`ForwardSlotsDirective`** (`forwardSlots()`) — Element directive that hands a host's slot content to a child element's SlotManager without any intermediate element.
- **`getSlotManager(host)`** — Returns or creates the SlotManager for a host element.

Content added between `connectedCallback()` and the first render is picked up when the host renders, and content added right after the first render is picked up by the observer without waiting for a macrotask. Rendering the slot conditionally is safe, including a container that is only rendered while `hasSlotContent()` is true, and a `slotContent()` part without a wrapping element. A component may switch its root template (like `pkt-heading` changing level) without its new template being taken for slot content.

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

```typescript
// In the radio and checkbox groups: null forwards the default slot, so the options land in the
// wrapper's options container without passing through an element of the group
render() {
  return html`<pkt-input-wrapper ${forwardSlots(this, [null, 'helptext'])} ...></pkt-input-wrapper>`
}
```

Rules:

- The child renders the forwarded nodes with its own `slotContent()`. The parent must not render the same slot with `slotContent()` as well.
- `null` in the array form forwards the default slot. Prefer it over rendering `${slotContent(this)}` inside the child's tag, which nests one Lit part inside another component's slotting.
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

### Framework anchors

Vue renders the host's children and keeps **anchor nodes** next to them: a `<!--v-if-->` comment where a conditional element was, and empty text nodes around fragments. When Vue later hides, shows or replaces a distributed element, it works through that element's _current_ parent, so the change lands inside the part instead of among the host's children.

The `SlotManager` that owns the content handles such changes (for forwarded slots, that is the host the content was given to, not the element rendering it):

- **Content** (elements, non-empty text) inserted inside the part is registered as slot content where it is, with a placeholder at the matching position among the host's children. It is not moved, so focus and state survive.
- **Anything else** (anchors) is moved to that position among the host's children, where Vue expects it. The anchor then survives the part being removed, for example when the container is only rendered while the slot has content.

The position comes from the placeholders of the nearest distributed nodes, or of a node removed from the same place in the same batch. A node inserted into a part that holds none of the host's content, with nothing removed there, is left alone.

The directive itself only ever inserts missing slot nodes before the part's end marker and leaves everything else in the part alone. Letting Lit commit the node list would clear the whole part on each change, including anchors, which made Vue crash with "Cannot read properties of null (reading 'insertBefore')" the next time the condition turned true.

A wrapper element in the consumer's markup (`<div><span v-if="…">…</span></div>` as the slotted node) is no longer needed to avoid this, but it does no harm.

Not handled: Vue reordering keyed `v-for` children that have been distributed. Vue inserts them relative to siblings that are no longer children of the host, which throws in the DOM before the `SlotManager` sees anything. Wrap such lists in one element.

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
