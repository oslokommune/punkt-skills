# Context API

For parent-child component communication, use Lit's `@lit/context` package. This is used when child components need to access data or methods from a parent without tight coupling.

## Example: pkt-tabs / pkt-tab-item

### Context definition

**File:** `src/components/tabs/tabs-context.ts`

```typescript
import { createContext } from '@lit/context'

export interface TabsContext {
  useArrowNav: boolean
  registerTab: (element: HTMLElement, index: number) => void
  handleClick: (index: number) => void
  handleKeyUp: (event: KeyboardEvent, index: number) => void
}

export const tabsContext = createContext<TabsContext>(Symbol('pkt-tabs-context'))
```

Use `Symbol()` to create a unique context key.

### Provider (parent component)

```typescript
import { provide } from '@lit/context'
import { tabsContext, type TabsContext } from './tabs-context'

@customElement('pkt-tabs')
export class PktTabs extends PktElement<IPktTabs> {

  @provide({ context: tabsContext })
  @state()
  private context: TabsContext = {
    useArrowNav: this.useArrowNav,
    registerTab: this.registerTab.bind(this),
    handleClick: this.handleClick.bind(this),
    handleKeyUp: this.handleKeyUp.bind(this),
  }

  // Update context when properties change
  updated(changedProperties: PropertyValues) {
    if (changedProperties.has('arrowNav') || changedProperties.has('disableArrowNav')) {
      this.context = {
        ...this.context,
        useArrowNav: this.useArrowNav,
      }
    }
  }
}
```

Key patterns:
- Use **`@provide()`** decorator with `@state()` (context must be reactive)
- **Bind methods** with `.bind(this)` in the context object
- **Spread and update** the context object when properties change (creates new reference to trigger consumers)

### Consumer (child component)

```typescript
import { consume } from '@lit/context'
import { tabsContext, type TabsContext } from './tabs-context'

@customElement('pkt-tab-item')
export class PktTabItem extends PktElement<IPktTabItem> {

  @consume({ context: tabsContext, subscribe: true })
  @property({ attribute: false })
  context?: TabsContext

  connectedCallback() {
    super.connectedCallback()
    this.updateComplete.then(() => {
      if (this.context) {
        this.context.registerTab(this.elementRef.value, this.index)
      }
    })
  }

  private handleClick() {
    this.context?.handleClick(this.index)
  }
}
```

Key patterns:
- Use **`@consume()`** with `subscribe: true` to get reactive updates
- Combine with `@property({ attribute: false })` — not exposed as HTML attribute
- **Optional type** (`context?`) — the consumer might render before context is available
- **Wait for `updateComplete`** before accessing context in `connectedCallback`

## When to use context

- **Parent-child coordination** — when children need to call methods on the parent
- **Shared state** — when multiple children need access to the same data
- **Decoupled communication** — when children shouldn't know the parent's implementation

## When NOT to use context

- **Simple props** — if a child just needs a value, use `@property`
- **Events** — if a child just needs to notify the parent, use events
- **Sibling communication** — context flows downward; for siblings, use events on a shared parent
