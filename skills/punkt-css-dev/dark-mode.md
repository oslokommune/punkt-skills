# Dark Mode

Dark mode is activated by the `data-mode="dark"` attribute on an ancestor element. The system works through CSS custom property overrides.

## How it works

1. Semantic color tokens are defined at `:root` for light mode
2. Those same tokens are redefined inside `[data-mode="dark"]`
3. Components reference `var(--pkt-color-*)` — they automatically adapt

```scss
// In base/_colors.scss
:root {
  --pkt-color-text-body-default: #2a2859;    // dark blue (light mode)

  [data-mode="dark"] {
    --pkt-color-text-body-default: #ffffff;   // white (dark mode)
  }
}
```

Because components use `var(--pkt-color-text-body-default)`, they get the correct color in both modes without any component-level overrides.

## When component-level overrides are needed

Sometimes a component needs additional dark mode adjustments beyond what the semantic tokens provide. Both patterns below are acceptable:

### Pattern 1: Nested selector (co-located)

Keeps the dark override next to the property it modifies:

```scss
.pkt-component {
  background: var(--pkt-color-background-card);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);

  [data-mode='dark'] & {
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.4);
  }
}
```

### Pattern 2: @at-root block (grouped)

Groups all dark overrides at the end of the component:

```scss
.pkt-component {
  background: var(--pkt-color-background-card);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

@at-root [data-mode='dark'] .pkt-component {
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.4);
}
```

Both are used in the codebase. Choose whichever fits the component best — the nested pattern works well for a few overrides, while the `@at-root` pattern is cleaner when there are many.

## Component-scoped dark tokens

For complex components with many color references, define component-specific CSS custom properties and override them in dark mode:

```scss
.pkt-btn {
  --pkt-color-button-text: var(--pkt-color-brand-dark-blue-1000);
  --pkt-color-button-bg: var(--pkt-color-brand-blue-1000);
  color: var(--pkt-color-button-text);
  background: var(--pkt-color-button-bg);
}

@at-root [data-mode='dark'] .pkt-btn {
  --pkt-color-button-text: var(--pkt-color-brand-neutrals-white);
  --pkt-color-button-bg: var(--pkt-color-brand-dark-blue-700);
}
```
