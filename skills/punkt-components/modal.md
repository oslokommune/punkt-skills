# Modal

Modal displays content in an overlay dialog or drawer that requires the user's attention. Use it for confirmations, detailed content, or actions that need focus without leaving the current page.

## Availability

| Package        | Available | Tag / Import                                                                                  |
| -------------- | --------- | --------------------------------------------------------------------------------------------- |
| React          | Yes       | `<PktModal>` — `import { PktModal } from '@oslokommune/punkt-react'`                          |
| Elements       | Yes       | `<pkt-modal>` — `import '@oslokommune/punkt-elements/dist/pkt-modal.js'`                      |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/latest/elements/pkt-modal.js" type="module">` |

Dark mode: No

## Variants

### Types

| Variant  | Use                                          |
| -------- | -------------------------------------------- |
| `dialog` | Centered overlay dialog (default)            |
| `drawer` | Side panel that slides in from left or right |

### Sizes

| Size               | Use                 |
| ------------------ | ------------------- |
| `small`            | Short confirmations |
| `medium` (default) | Standard dialogs    |
| `large`            | Detailed content    |

### Close button skins

| Skin            | Description          |
| --------------- | -------------------- |
| `blue`          | Default close button |
| `yellow-filled` | Yellow close button  |

## Usage guidelines

**Use modal when:**

- You need to display detailed content without taking the user out of context
- You need a confirmation before a destructive or important action
- You want the user to focus on a specific task or form

**Avoid modal when:**

- The content can be shown inline on the page
- You only need to show a short message — use Alert or Messagebox instead

## Props / Attributes

| Prop (React)           | Attribute (Elements)   | Type                                 | Default    | Description                                                  |
| ---------------------- | ---------------------- | ------------------------------------ | ---------- | ------------------------------------------------------------ |
| `open`                 | `open`                 | boolean                              | `false`    | Declarative open/close control — use instead of ref methods  |
| `headingText`          | `headingText`          | string                               | —          | Heading text for the modal                                   |
| `hideCloseButton`      | `hideCloseButton`      | boolean                              | `false`    | Hides the close button — provide an alternative way to close |
| `closeOnBackdropClick` | `closeOnBackdropClick` | boolean                              | `false`    | Close modal when clicking the backdrop                       |
| `closeButtonSkin`      | `closeButtonSkin`      | `"blue"` \| `"yellow-filled"`        | `"blue"`   | Style of the close button                                    |
| `size`                 | `size`                 | `"small"` \| `"medium"` \| `"large"` | `"medium"` | Modal size                                                   |
| `variant`              | `variant`              | `"dialog"` \| `"drawer"`             | `"dialog"` | Modal type                                                   |
| `drawerPosition`       | `drawerPosition`       | `"left"` \| `"right"`                | `"right"`  | Position of the drawer (when variant is `"drawer"`)          |
| `transparentBackdrop`  | `transparentBackdrop`  | boolean                              | `false`    | Make the backdrop transparent                                |
| `removePadding`        | `removePadding`        | boolean                              | `false`    | Remove inner padding from the modal content                  |

## Events

| Event (React)       | Event (Elements)   | Description                        |
| ------------------- | ------------------ | ---------------------------------- |
| `onClose`           | `close`            | Fires when the modal is closed     |
| `onBackgroundClick` | `background-click` | Fires when the backdrop is clicked |

## Slots

| Slot    | Description                                                            |
| ------- | ---------------------------------------------------------------------- |
| default | Modal content. CSS helper classes are provided for styling 1–3 buttons |

## Declarative vs imperative — pick one

PktModal supports two ways to open/close:

1. **Declarative** (recommended): Use the `open` prop/attribute + `onClose`/`close` event to manage state
2. **Imperative**: Call `showModal()` / `close()` directly (via `ref` in React, or on the element in Elements)

**Do not mix both approaches on the same modal.** The native `<dialog>` element does not fire an event when opened — only when closed. If you use the `open` prop to manage state but also call `showModal()` imperatively, your state will be out of sync and there is no `onOpen` event to catch it.

### React
Use `open` + `onClose` exclusively, or `ref` exclusively. Do not combine them.

### Elements / HTML + JavaScript
Use the `open` attribute + `close` event listener exclusively, or call `.showModal()` / `.close()` imperatively.

**Declarative** — toggle the attribute and listen for `close`:
```html
<pkt-modal id="myModal" headingText="Title" open>...</pkt-modal>
<script>
  const modal = document.querySelector('#myModal')
  // Close: remove the attribute
  modal.removeAttribute('open')
  // Open: set the attribute
  modal.setAttribute('open', '')
  // Listen for close (Esc, close button, backdrop)
  modal.addEventListener('close', () => { ... })
</script>
```

**Imperative** — call methods directly:
```html
<pkt-modal id="myModal" headingText="Title">...</pkt-modal>
<script>
  const modal = document.querySelector('#myModal')
  modal.showModal()
  modal.addEventListener('close', () => { ... })
</script>
```

Do not mix — e.g., don't set the `open` attribute and also call `.showModal()` on the same element.

## Accessibility

- The modal traps focus when open — Tab cycles through focusable elements inside the modal
- Close the modal with Esc
- If `hideCloseButton` is true, you must provide an alternative way to close the modal
- The heading should be clear and descriptive

## Testing

PktModal uses the native `<dialog>` element with `showModal()`. **jsdom does not support `HTMLDialogElement`** ([jsdom#3294](https://github.com/jsdom/jsdom/issues/3294)) — `showModal()`, `close()`, and the `::backdrop` pseudo-element are not implemented.

This means:
- Tests that call `showModal()` or `close()` (directly or via the `open` prop) will fail in jsdom
- The `open` attribute on `<dialog>` works in jsdom, but `showModal()` does not (they behave differently — `showModal()` adds backdrop and traps focus)

**Workarounds:**

1. **Mock `showModal` and `close`** in your test setup:
   ```ts
   beforeEach(() => {
     HTMLDialogElement.prototype.show = vi.fn(function (this: HTMLDialogElement) {
       this.setAttribute('open', '')
     })
     HTMLDialogElement.prototype.showModal = vi.fn(function (this: HTMLDialogElement) {
       this.setAttribute('open', '')
     })
     HTMLDialogElement.prototype.close = vi.fn(function (this: HTMLDialogElement) {
       this.removeAttribute('open')
     })
   })
   ```

2. **Use a real browser environment** like Playwright or Cypress for integration tests that need full dialog behavior.

3. **Limit unit tests** to rendering, prop application, and CSS class generation — avoid testing open/close lifecycle in jsdom.

## Examples

### React (declarative with `open` prop — recommended)

```jsx
import { useState } from 'react'
import { PktModal, PktButton } from '@oslokommune/punkt-react'

const [isOpen, setIsOpen] = useState(false)

{/* Confirmation dialog */}
<PktButton onClick={() => setIsOpen(true)}>Delete item</PktButton>
<PktModal open={isOpen} headingText="Delete item?" size="small" onClose={() => setIsOpen(false)}>
  <p>This action cannot be undone.</p>
  <div className="pkt-modal__buttons">
    <PktButton color="red" onClick={handleDelete}>Delete</PktButton>
    <PktButton skin="tertiary" onClick={() => setIsOpen(false)}>Cancel</PktButton>
  </div>
</PktModal>

{/* Drawer */}
<PktModal
  open={drawerOpen}
  headingText="Filters"
  variant="drawer"
  drawerPosition="right"
  onClose={() => setDrawerOpen(false)}
>
  <p>Filter options here.</p>
</PktModal>
```

### React (imperative with ref — still supported)

```jsx
import { useRef } from 'react'
import { PktModal, PktButton } from '@oslokommune/punkt-react'

const dialogRef = useRef<HTMLDialogElement>(null)

<PktButton onClick={() => dialogRef.current?.showModal()}>Open</PktButton>
<PktModal ref={dialogRef} headingText="Title" onClose={() => console.log('closed')}>
  <p>Content</p>
</PktModal>
```

### Elements

```html
<!-- Declarative -->
<pkt-modal open headingText="Delete item?" size="small">
  <p>This action cannot be undone.</p>
  <div class="pkt-modal__buttons">
    <pkt-button color="red"><span>Delete</span></pkt-button>
    <pkt-button skin="tertiary"><span>Cancel</span></pkt-button>
  </div>
</pkt-modal>

<!-- Imperative -->
<pkt-modal id="myModal" headingText="Delete item?" size="small">
  ...
</pkt-modal>
<script>
  document.querySelector('#myModal').showModal()
  document.querySelector('#myModal').addEventListener('close', () => { ... })
</script>
```
