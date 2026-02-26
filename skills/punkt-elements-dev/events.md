# Events

## Event conventions

All events in Punkt elements:
- **Bubble** (`bubbles: true`)
- **Compose** (`composed: true`) — crosses shadow DOM boundaries
- Use standard `Event` or `CustomEvent` constructors

## Standard form events (PktInputElement)

The base class provides helper methods that dispatch standard events:

```typescript
// Dispatches both change and value-change events
protected dispatchChangeEvents(value: unknown): void {
  this.dispatchEvent(new Event('change', { bubbles: true, composed: true }))
  this.dispatchEvent(new CustomEvent('value-change', {
    detail: value,
    bubbles: true,
    composed: true,
  }))
}

// Focus/blur
protected onFocus(): void {
  this.dispatchEvent(new FocusEvent('focus', { bubbles: true, composed: true }))
}

protected onBlur(): void {
  this.dispatchEvent(new FocusEvent('blur', { bubbles: true, composed: true }))
}

// Input
protected onInput(): void {
  this.dispatchEvent(new InputEvent('input', { bubbles: true, composed: true }))
}
```

## Custom events

For component-specific events, use `CustomEvent` with a descriptive name and `detail` payload:

```typescript
this.dispatchEvent(
  new CustomEvent('tab-selected', {
    detail: { index },
    bubbles: true,
    composed: true,
  })
)
```

```typescript
this.dispatchEvent(
  new CustomEvent('close', {
    detail: { origin: event },
    bubbles: true,
    composed: true,
  })
)
```

## Event stopping in inputs

When handling native input events inside a component's render method, always **stop immediate propagation** to prevent the event from firing twice (once from the native element, once re-dispatched by the component):

```typescript
@change=${(e: Event) => {
  this.touched = true
  this.value = (e.target as HTMLInputElement).value
  e.stopImmediatePropagation()  // Prevent duplicate
}}
@input=${(e: InputEvent) => {
  this.onInput()
  e.stopImmediatePropagation()
}}
@focus=${(e: FocusEvent) => {
  this.onFocus()
  e.stopImmediatePropagation()
}}
@blur=${(e: FocusEvent) => {
  this.onBlur()
  e.stopImmediatePropagation()
}}
```

## Click prevention (disabled/loading)

Components that can be disabled or in a loading state prevent clicks at the element level:

```typescript
connectedCallback(): void {
  super.connectedCallback()
  this.addEventListener('click', (e) => {
    if (this.disabled || this.isLoading) {
      e.preventDefault()
      e.stopImmediatePropagation()
    }
  }, true)  // Capture phase

  this.addEventListener('keydown', (e) => {
    if (!(this.disabled || this.isLoading)) return
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault()
      e.stopImmediatePropagation()
    }
  }, true)
}
```

Note the `true` parameter for capture phase — this ensures the event is caught before any other listeners.

## Event binding in templates

Use Lit's `@event` syntax:

```typescript
html`
  <button @click=${this.handleClick}>Click</button>
  <input @change=${(e: Event) => this.handleChange(e)} />
`
```

For methods that need `this` context, either use arrow functions in the template or bind in the constructor:

```typescript
// Arrow function in template (common)
@click=${() => this.handleClick()}

// Or bind in class (for methods referenced by name)
private handleClick = () => { ... }
```
