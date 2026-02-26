# Accessibility

## Screen reader content

Use the `%sr-only` placeholder for visually hidden but accessible content:

```scss
.pkt-component__sr-label {
  @extend %sr-only;
}
```

Or use the utility class in HTML:
```html
<span class="pkt-sr-only">Screen reader text</span>
```

## Focus indicators

All interactive elements must have visible focus indicators:

1. **`:focus-visible`** for keyboard navigation — double box-shadow ring
2. **`:focus:not(:focus-visible)`** for mouse clicks — simpler outline

See [States & Interactivity](states.md) for the exact pattern.

## Reduced motion

The normalize layer includes a global `prefers-reduced-motion` rule:

```scss
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

Components with custom animations should respect this preference:

```scss
.pkt-component {
  transition: opacity 200ms ease;

  @media (prefers-reduced-motion: reduce) {
    transition: none;
  }
}
```

## Color contrast

- Use semantic color tokens (`--pkt-color-text-*`, `--pkt-color-background-*`) which are designed to meet WCAG contrast ratios
- Dark mode tokens are also contrast-verified
- Do not use brand colors directly for text on arbitrary backgrounds without checking contrast
