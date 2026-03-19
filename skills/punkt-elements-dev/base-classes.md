# Base Classes

All base classes live in `src/base-elements/`. Every Punkt element extends one of these.

## PktShadowElement

**File:** `src/base-elements/element.ts`
**Extends:** `LitElement`
**DOM:** Shadow DOM (default Lit behavior)

The root base class. Provides:

- **`strings`** property — Norwegian translations from `translations/no.json` (validation messages, form labels)
- **`role`** property — ARIA role attribute
- **Index signature** `[key: string]: any` — allows dynamic property access (needed for controllers)
- **`$props`** — Vue prop typing support
- **`hotReplacedCallback()`** — HMR support in dev mode

```typescript
import { LitElement } from 'lit'
import { property } from 'lit/decorators.js'
import strings from '../translations/no.json'

export class PktShadowElement<T = {}> extends LitElement {
  [key: string]: any

  @property({ type: Object }) strings: any = strings
  @property({ type: String }) role: string | null = null

  $props!: T & Props

  hotReplacedCallback() {
    this.requestUpdate()
  }
}
```

**When to use:** Only when Shadow DOM encapsulation is explicitly needed (rare). Currently used by `pkt-heading` and `pkt-accordion` (container).

## PktElement

**File:** `src/base-elements/element.ts`
**Extends:** `PktShadowElement`
**DOM:** Light DOM

The standard base class for most components. Overrides `createRenderRoot()` to render into the host element itself instead of a shadow root.

```typescript
export class PktElement<T = {}> extends PktShadowElement<T> {
  createRenderRoot() {
    return this
  }
}
```

**When to use:** For light DOM components that do NOT accept slot content (e.g., icon, checkbox, radiobutton, progressbar, calendar, consent, backlink).

## PktElementWithSlot

**File:** `src/base-elements/element.ts`
**Extends:** `PktElement`
**DOM:** Light DOM

Extends `PktElement` with slot content support. Pre-collects children in `connectedCallback()` before Lit's first render clears them, and provides `hasSlotContent()`. Only components that use the `slotContent` directive should extend this class — components without slots use plain `PktElement` to avoid unnecessary overhead.

```typescript
export class PktElementWithSlot<T = {}> extends PktElement<T> {
  override connectedCallback(): void {
    getSlotManager(this).collectNodes()
    super.connectedCallback()
  }

  hasSlotContent(slotName?: string): boolean {
    return getSlotManager(this).hasContent(slotName)
  }
}
```

**When to use:** For components whose template uses `slotContent(this)` or `slotContent(this, 'name')`. Used by alert, button, card, link, tag, accordion-item, tab-item, tabs, header, modal, loader, messagebox, helptext, input-wrapper, linkcard.

## PktInputElement

**File:** `src/base-elements/input-element.ts`
**Extends:** `PktElementWithSlot`
**DOM:** Light DOM

Base class for all form input components. Provides full form participation via `ElementInternals`. Extends `PktElementWithSlot` because most input components forward helptext slot content. See [Form Integration](form-integration.md) for detailed documentation.

**When to use:** For any component that participates in HTML forms (text inputs, checkboxes, radio buttons, textareas).

## PktOptionsInputElement

**File:** `src/base-elements/options-input-element.ts`
**Extends:** `PktInputElement`
**DOM:** Light DOM

Base class for inputs with selectable options. Manages option state from both props and slot `<option>` elements. See [Form Integration](form-integration.md) for detailed documentation.

**When to use:** For components that have selectable options (select, combobox, listbox).

## Choosing the right base class

| Scenario | Base Class |
|---|---|
| Display component with slot content (alert, card, link, tag, etc.) | `PktElementWithSlot` |
| Display component without slot content (icon, progressbar, etc.) | `PktElement` |
| Component needing Shadow DOM encapsulation | `PktShadowElement` |
| Text input, checkbox, radio, textarea | `PktInputElement` |
| Select, combobox, listbox | `PktOptionsInputElement` |

## Generic type parameter

All base classes accept a generic `<T>` parameter used for the `$props` typing (Vue integration):

```typescript
@customElement('pkt-button')
export class PktButton extends PktElementWithSlot<IPktButton> implements IPktButton {
  // ...
}
```

Pass the component's public interface as `T` and `implements` the same interface to ensure the class satisfies its API contract.
