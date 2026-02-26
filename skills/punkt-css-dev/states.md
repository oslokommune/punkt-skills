# States & Interactivity

## Interactive state pattern

All interactive components provide both pseudo-class states and corresponding BEM modifier classes (for documentation/testing):

```scss
.pkt-btn {
  // Normal
  color: var(--pkt-color-button-text-normal);
  background: var(--pkt-color-button-background-normal);

  // Hover
  &:hover,
  &.pkt-btn--hover {
    color: var(--pkt-color-button-text-hover);
    text-decoration: underline;
  }

  // Focus (keyboard only)
  &:focus-visible,
  &.pkt-btn--focus {
    box-shadow:
      0px 0px 0px 0.125rem var(--pkt-color-text-action-active),
      0px 0px 0px 0.375rem var(--pkt-color-border-states-focus);
    outline: none;
  }

  // Focus (mouse click fallback)
  &:focus:not(:focus-visible) {
    outline: 0.125rem solid var(--pkt-color-text-action-active);
  }

  // Active
  &:active,
  &.pkt-btn--active {
    color: var(--pkt-color-button-text-active);
  }

  // Disabled
  &:disabled,
  &[disabled],
  &[data-disabled] {
    color: var(--pkt-color-button-text-disabled);
    cursor: inherit;
    pointer-events: none;
  }
}
```

## Focus ring technique

The standard focus ring uses a **double box-shadow** (inner ring + outer ring):

```scss
&:focus-visible {
  box-shadow:
    0px 0px 0px 0.125rem var(--pkt-color-text-action-active),   // Inner ring (2px)
    0px 0px 0px 0.375rem var(--pkt-color-border-states-focus);   // Outer ring (6px)
  outline: none;
}
```

For non-`:focus-visible` focus (mouse clicks), use a simpler outline:
```scss
&:focus:not(:focus-visible) {
  outline: 0.125rem solid var(--pkt-color-text-action-active);
}
```

## Disabled state

Always handle all three disabled variants:

```scss
&:disabled,
&[disabled],
&[data-disabled] {
  // Muted colors
  cursor: inherit;
  pointer-events: none;
}
```

- `:disabled` — native HTML disabled attribute (form elements)
- `[disabled]` — attribute selector (works on any element)
- `[data-disabled]` — data attribute (used by web components)
