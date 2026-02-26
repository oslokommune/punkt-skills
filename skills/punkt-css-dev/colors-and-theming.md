# Colors & Theming

## Three-tier color system

### 1. Brand colors (`$pkt-colors`) — raw values

55 tokens representing the visual identity. These do not change between light and dark mode.

Categories:
- **Blues**: `brand-blue-100` through `brand-blue-1000`, `brand-dark-blue-700/1000`, `brand-warm-blue-1000`
- **Greens**: `brand-green-400/1000`, `brand-light-green-400/1000`, `brand-dark-green-1000`
- **Yellows**: `brand-yellow-400/1000`
- **Reds**: `brand-red-400/700/1000`
- **Beiges**: `brand-beige-400/700/1000`
- **Neutrals**: `brand-neutrals-white`, `brand-neutrals-100`, `brand-neutrals-200`, `brand-neutrals-1000`, `brand-neutrals-black`, `brand-neutrals-transparent`
- **Grays**: `grays-gray-100` through `grays-gray-1000`

### 2. Semantic colors (`$pkt-semantic-colors`) — purpose-mapped

54 tokens that map to brand colors and change between light/dark mode. **Always prefer semantic colors in component styles.**

Categories:
- `background-*` — page/card/section backgrounds
- `border-*` — borders, dividers, input borders
- `surface-*` — surface colors (cards, panels)
- `text-*` — body text, headings, actions, links
- `input-*` — form input states (normal, hover, active, disabled)

### 3. Deprecated colors (`$colors`) — legacy

40+ tokens with old naming (`color-blue`, `color-green`, etc.). **Do not use in new code.**

## CSS custom properties

All colors are exported as CSS custom properties at `:root`:

```scss
:root {
  // Brand (constant)
  --pkt-color-brand-blue-1000: #6fe9ff;
  --pkt-color-brand-dark-blue-1000: #2a2859;

  // Semantic (changes in dark mode)
  --pkt-color-text-body-default: #2a2859;
  --pkt-color-background-default: #ffffff;
}
```

## Using colors in components

Always reference CSS custom properties, not Sass variables directly:

```scss
// Correct — uses CSS custom property, responds to dark mode
.pkt-component {
  color: var(--pkt-color-text-body-default);
  background: var(--pkt-color-background-card);
  border: 1px solid var(--pkt-color-border-default);
}

// Wrong — hardcoded Sass value, ignores dark mode
.pkt-component {
  color: map.get(variables.$pkt-semantic-colors, 'text-body-default');
}
```

## Component-specific tokens

Components can define their own CSS custom properties scoped to the block, then override in modifiers:

```scss
.pkt-btn {
  background-color: var(--pkt-color-button-background-normal);
  border-color: var(--pkt-color-button-border-normal);
  color: var(--pkt-color-button-text-normal);
}

.pkt-btn--green {
  --pkt-color-button-background-normal: var(--pkt-color-brand-green-1000);
  --pkt-color-button-border-normal: var(--pkt-color-brand-green-1000);
  --pkt-color-button-text-normal: var(--pkt-color-brand-dark-blue-1000);
}
```

## Utility classes

Generated for all brand + semantic colors:
- `.pkt-color-bg-{name}` — background-color
- `.pkt-color-txt-{name}` — color
- `.pkt-color-border-{name}` — border-color
