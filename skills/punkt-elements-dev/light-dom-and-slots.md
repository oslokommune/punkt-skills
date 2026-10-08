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

The Elements dev app has an illustrated walkthrough in Norwegian at `/slik-virker-slots` (`src/docs/components/slot-internals.ts`): host layout, a connection map per scenario, the content end, whitespace mirrors and forwarding. Keep its file map in step when sections or functions in `slot-content.ts` change.

### How it works

1. Components that use slots extend `PktElementWithSlot` (instead of `PktElement`)
2. `PktElementWithSlot.connectedCallback()` calls `SlotManager.collectNodes()` to capture children before Lit renders. Right before the first render, the `SlotManager` marks the end of the content (see _Host layout_)
3. The component's template uses `${slotContent(this)}` to place default slot content
4. Named slots use `${slotContent(this, 'slotName')}`
5. A `MutationObserver` watches the host's subtree for as long as the host is connected. New content counts when it is added as a direct child of the host, or when other code inserts it inside a part that holds the host's content (see _Framework anchors_). Removal counts wherever the node sits, including after it has been distributed into the template. Changes made while the host is disconnected are caught up on when it connects again (see _Disconnected hosts_)
6. A reactive controller on the host separates Lit's own render from everybody else's changes: pending mutations are handled right before the host renders, and nodes Lit adds or removes while rendering are never taken for slot content changes
7. `PktElementWithSlot` overrides the host's DOM methods (`appendChild`, `insertBefore`, `removeChild`, `textContent` and others), so changes made through them reach distributed content and are handled at once (see _DOM methods on the host_)
8. Every slot change calls `requestUpdate()` on the host, so anything computed from slot content in `render()` (like `hasSlotContent()`) stays reactive in both directions: none → some and some → none
9. The directive uses a generation counter to avoid unnecessary DOM updates

### Architecture

- **`SlotManager`** — Per-host singleton (stored in a `WeakMap`). Collects and categorizes children by `slot` attribute, observes mutations, notifies registered directives, and notifies other managers that receive its slots via `forwardSlots()`.
- **`SlotContentDirective`** — `AsyncDirective` that moves the collected nodes into its part itself and always returns `noChange`. It never hands Lit the node list: Lit would then clear everything between the part markers on each change, including nodes it did not put there (see _Framework anchors_ below).
- **`ForwardSlotsDirective`** (`forwardSlots()`) — Element directive that hands a host's slot content to a child element's SlotManager without any intermediate element.
- **`getSlotManager(host)`** — Returns or creates the SlotManager for a host element.

### Host layout

The host's children come in this order:

```text
<pkt-button>
  <!--pkt-slot--> …                    content: placeholders, anchors, whitespace, options
  <!---->                              content end
  <!---->                              Lit's root marker
  <button class="pkt-btn">…</button>   the template
</pkt-button>
```

The `SlotManager` appends the content end comment right before the host's first render, so Lit renders the template after it. Everything before it is the host's content:

- Content appended with `appendChild`, `append`, or a framework's insert with no reference node (`null`), goes at the end of the content, before the template. A reference node in the template means the end of the content too. Placeholders always go in the content, so a component that switches its root template (`pkt-heading` changing level) never removes them.
- `replaceContent()` empties the whole content, like the native setters do.

The content end comment reports `nextSibling` as `null`. A Lit parent with no whitespace before the closing tag (`<pkt-button>${label}</pkt-button>`) has a part that runs to the end of the host, and when the value becomes empty or changes type, Lit clears it by walking `nextSibling` until `null`. The walk stops at the content end, so the template is never removed: focus, an open `<dialog>` and the template's own elements stay as they are. The clear does remove the content end comment itself, as the last node it reaches. `getContentEnd()` puts it back before Lit's root marker the next time it is needed. The `SlotManager` reads the real next sibling through `Node.prototype` when it needs it.

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

Slot content can be bare text or elements, and it can change after the component has rendered. No wrapper element is needed:

```html
<pkt-alert>Hei fra ${this.name}</pkt-alert>
<pkt-button>${this.label}</pkt-button>
<pkt-alert
  >${repeat(this.items, (i) => i.id, (i) => html`
  <p>${i.text}</p>
  `)}</pkt-alert
>
```

For several items, like a list, recommend one wrapper element anyway (`<div>`, `<ul>`). Changes inside a wrapper are plain DOM, while changes to the host's own children go through the `SlotManager`. Measured in Chromium (Lit dev build): 1000 synchronous `appendChild` calls take about 15 ms on the host and 0.7 ms inside a `<div>`; a Vue keyed list of 300 items shuffled 20 times takes about 36 ms directly in the component and 14 ms inside a `<div>`.

A Lit parent updates a text binding by writing to the node after its marker, which is the placeholder once the text node has been distributed. Placeholders for text nodes forward writes to `data` to the text node. When Lit's `repeat()` reorders items, it moves the placeholders, and the distributed content follows. A binding with no whitespace before the closing tag (`<pkt-tag>${text}</pkt-tag>`) is safe too, because a clear stops at the content end (see _Host layout_).

Frameworks change text through `data` (Lit, Preact) or `nodeValue` (Vue, React, Angular), which the observer does not report. `watchText()` therefore gives text nodes in the host's content and distributed text nodes their own `data` and `nodeValue` accessors:

- Empty text in the host (Vue keeps one for `{{ text }}` while the value is empty) becomes slot content when it gets text.
- Distributed text that becomes empty or whitespace is released: it goes back to its placeholder's position among the host's children, `hasSlotContent()` turns false, and it becomes slot content again when it gets text.

### Whitespace

Whitespace-only text at the start and the end of the content is left out, and whitespace between two pieces of default slot content is shown: `<pkt-alert> <b>Hei</b> <i>du</i> </pkt-alert>` renders "Hei du", and so does React's `'Hei ', name, ' ', <b/>`, where every string is its own text node. Whitespace with only named slot content, `<option>`s or anchors on one side counts as start or end. Text with any other characters is content and keeps its whitespace.

The whitespace node itself stays among the host's children, because a Lit parent may use it as the end of a part (`${a} ${b}`): moving it into the template would make clearing `a` remove `b` too. `syncWhitespace()` gives the slot a text node of its own, a mirror, at that position instead. The whitespace node acts as the mirror's placeholder, so the mirror follows when it moves and goes when it is removed, and changes to its text reach the mirror. `syncWhitespace()` runs on every slot change, but only looks where something can have changed: mirrors that are no longer between content sit at the ends of the default slot's list (which is in the order of the host's children), and new candidates are whitespace that was added or changed, and whitespace where the first or last content moved. A full scan on every change made 1000 appends with spaces between them take 2.8 s and 1000 removals from the end 5.5 s; now they take about 70 ms and 30 ms.

Mirrors are placed with the content (`placementNodes()`), but they are not content: `getNodes()` and `hasSlotContent()` leave them out.

### Framework anchors

Vue renders the host's children and keeps **anchor nodes** next to them: a `<!--v-if-->` comment where a conditional element was, and empty text nodes around fragments. When Vue later hides, shows or replaces a distributed element, it works through that element's _current_ parent, so the change lands inside the part instead of among the host's children.

The `SlotManager` that owns the content handles such changes (for forwarded slots, that is the host the content was given to, not the element rendering it):

- **Content** (elements, non-empty text) inserted inside the part is registered as slot content where it is, with a placeholder at the matching position among the host's children. It is not moved, so focus and state survive.
- **Anything else** (anchors) is moved to that position among the host's children, where Vue expects it. The anchor then survives the part being removed, for example when the container is only rendered while the slot has content.

The position comes from the placeholders of the nearest distributed nodes, or of a node removed from the same place in the same batch. A node inserted into a part that holds none of the host's content, with nothing removed there, is left alone, so decoration a component adds to its own part (like the separators in `pkt-tabs`) is not taken for slot content.

The directive itself only ever inserts missing slot nodes before the part's end marker and leaves everything else in the part alone. Letting Lit commit the node list would clear the whole part on each change, including anchors, which made Vue crash with "Cannot read properties of null (reading 'insertBefore')" the next time the condition turned true.

### DOM methods on the host

Vue, React and plain JavaScript change the host's children through the host's DOM methods, but distributed content is no longer a child of the host. `PktElementWithSlot` therefore overrides them:

| Method                                                               | Behavior                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `appendChild`, `append`                                              | Insert at the end of the content, before the template, and handle the change at once: the content is in its slot when the call returns                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `prepend`                                                            | Native, and the change is handled at once                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `insertBefore`                                                       | A distributed reference node stands at its placeholder, so keyed reordering (Vue `v-for`, React `key`) works. A reference elsewhere in a part, like the node after the last distributed node (`ref.nextSibling`, or Vue's anchor when it swaps an element), stands next to the nearest distributed node, or at the end when the part holds none. A reference in the template means the end of the content. Moving a node that is already distributed (also with `appendChild`) is done in place by `SlotManager.moveDistributed()`, without placing the whole slot again |
| `removeChild`, `replaceChild`                                        | Also accept a distributed child, instead of throwing `NotFoundError`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `textContent`, `innerHTML`, `innerText` (setters), `replaceChildren` | Replace the host's content (`SlotManager.replaceContent()`) and leave the rendered template alone. Vue and React set `textContent` to update text-only children (`<pkt-button>{{ label }}</pkt-button>`), and `v-html` / `dangerouslySetInnerHTML` set `innerHTML`. `innerText` turns line breaks into `<br>` like the native setter, and works like `textContent` where the environment has no `innerText` (jsdom)                                                                                                                                                      |

While the host renders, the methods are native: Lit inserts its own template through `insertBefore`. A microtask clears the render flag as well, since `hostUpdated` is skipped when `render()` throws. Before the children have been collected, the setters are native too, and so is `replaceChildren` when a node contains the host, so that it throws before changing anything. The getters read natively, so `innerHTML` still reads the full rendered markup; the `innerText` getter falls back to `textContent` where the environment has no `innerText`. The `SlotManager` uses the native methods for its own DOM changes, and runs its own changes through `asOwnChange()`, which drops the observer records they cause and restores the guard flag even if a step throws.

### Disconnected hosts

The observer stops while the host is disconnected, so changes made then are not reported. When the host connects again, `SlotManager.resync()` catches up: content appended meanwhile is distributed, content removed meanwhile (or whose placeholder was removed, like by a Lit parent) is dropped instead of put back, and placeholders that were moved (Lit's `repeat()`) reorder the distributed nodes. That covers, for example, a cached view that is updated while it is detached. `replaceContent()` and `moveDistributed()` work while disconnected without waiting for `resync()`.

## Automatic filtering

The `slotContent` directive **always** filters out:

- `<option>` and `<data>` elements (handled by `PktOptionsSlotController`)
- Elements with the `data-skip` attribute
- Elements injected by the dialog polyfill (`_dialog_overlay`, `backdrop`)
- Whitespace-only text at the start and the end of the content (see _Whitespace_)

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
