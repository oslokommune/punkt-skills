# Colors & Theming

## Three-tier color system

### 1. Brand colors (`$pkt-colors`) — raw values

46 tokens representing the visual identity. These do not change between light and dark mode.

Categories:
- **Blues**: `brand-blue-100/200/300/500/1000`, `brand-dark-blue-700/1000`, `brand-warm-blue-1000`
- **Greens**: `brand-green-400/1000`, `brand-light-green-400/1000`, `brand-dark-green-1000`
- **Yellows**: `brand-yellow-500/1000`
- **Reds**: `brand-red-100/400/600/1000`
- **Beiges**: `brand-light-beige-1000`, `brand-dark-beige-1000`
- **Purple**: `brand-purple-1000`
- **Neutrals**: `brand-neutrals-white`, `brand-neutrals-50` through `brand-neutrals-1000` (50, 100,
  200, 300, 400, 500, 600, 700, 800, 900, 1000), `brand-neutrals-black`,
  `brand-neutrals-transparent`

The neutral scale runs light to dark with ascending numbers.

**Deprecated:** `grays-gray-100` through `grays-gray-1000` still resolve, but are superseded by the
neutral scale. Map old to new by value, not by number:

| Deprecated | Use instead |
| --- | --- |
| `grays-gray-100` | `brand-neutrals-200` |
| `grays-gray-200` | `brand-neutrals-300` |
| `grays-gray-300` | `brand-neutrals-400` |
| `grays-gray-400` | `brand-neutrals-500` |
| `grays-gray-500` | `brand-neutrals-600` |
| `grays-gray-600` | `brand-neutrals-700` |
| `grays-gray-700` | `brand-neutrals-800` |
| `grays-gray-800` | `brand-neutrals-900` |
| `grays-gray-1000` | `brand-neutrals-1000` |

### 2. Semantic colors — purpose-mapped

104 tokens that map to brand colors. **Always prefer semantic colors in component styles.**

Two maps, one source:

**`$pkt-semantic-color-modes`** is where tokens are defined. Each entry is a `(light, dark)` pair of
names from `$pkt-colors`; a `null` dark value means the token is identical in both modes. Both modes
are generated from this map in `base/_colors-tokens.scss`, so a token cannot drift between them:

```scss
$pkt-semantic-color-modes: (
  // identical in both modes
  surface-strong-blue: (brand-blue-1000, null),
  // flips in dark mode
  text-body-default: (brand-dark-blue-1000, brand-neutrals-white),
);
```

When adding a token, add it here and decide the dark value at the same time — that is the point of
the pair.

**`$pkt-semantic-colors`** and **`$pkt-semantic-colors-dark`** are the public maps, derived from the
above: `token -> colour`, one per mode. Both contain every token — where the source has a `null` dark
value, the dark map holds the light colour, so a lookup always returns a colour. Both keep the shape
`$pkt-semantic-colors` has always had, so reading them with `map.get()` or configuring them with
`@use ... with ()` works as before. Do not add tokens to these; they are generated.

```scss
@use 'sass:map';
@use '@oslokommune/punkt-css/dist/scss/abstracts/variables';

$light: map.get(variables.$pkt-semantic-colors, 'text-body-default'); // #2a2859
$dark: map.get(variables.$pkt-semantic-colors-dark, 'text-body-default'); // #ffffff
```

`base/_colors-tokens.scss` emits the light map on `:root`, then emits from the dark map only the
tokens whose value actually differs — so overriding either map takes effect.

Categories:
- `background-*` — page/card/section backgrounds
- `border-*` — borders, dividers, input borders
- `surface-*` — surface colors (cards, panels). These no longer flip in dark mode
- `text-*` — body text, headings, actions, links
- `input-*` — form input states (normal, hover, active, open, disabled)
- `tag-*` — tag text states
- `button-{primary,secondary,tertiary}-*` — button variants

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

### `functions.color-token()` — the preferred way

Use the helper rather than writing `var()` by hand. It emits the custom property with the map value
as a fallback, so the fallback can never go stale, and an unknown token name fails the build instead
of silently producing an invalid declaration.

```scss
@use '../abstracts/functions';

.pkt-component {
  color: functions.color-token('text-body-default');
  // => color: var(--pkt-color-text-body-default, #2a2859);
}
```

It looks up `$pkt-semantic-colors` first, then falls back to `$pkt-colors`, so both semantic tokens
and brand colours work — **prefer a semantic token whenever one exists.** Inside an interpolated
context (a `var()` fallback, a gradient, a mixin argument) wrap it: `#{functions.color-token('…')}`.

The fallback is always the light value. If the custom properties are unavailable, dark mode gets
light colours — unavoidable, and no worse than a hardcoded literal.

### Referencing colours directly

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

`elements/_button.scss` keeps its own `$-skins` map, which resolves these generic
`--pkt-color-button-*` properties per skin and per mode. The `primary`, `secondary` and `tertiary`
skins point at the global `button-{variant}-*` semantic tokens. The colour variants (green, blue,
beige, yellow, red) have no design tokens of their own and reference brand colors directly.

## Utility classes

Generated for all brand + semantic colors:
- `.pkt-color-bg-{name}` — background-color
- `.pkt-color-txt-{name}` — color
- `.pkt-color-border-{name}` — border-color
