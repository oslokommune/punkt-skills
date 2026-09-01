# Abstracts (Variables, Mixins, Functions)

All shared abstracts live in `src/scss/abstracts/` and are imported with namespaced `@use`:

```scss
@use 'sass:map';
@use '../abstracts/variables';
@use '../abstracts/mixins/breakpoints' as *;
@use '../abstracts/mixins/typography';
```

**Always use namespaced imports** (e.g., `variables.$spacing`, `typography.get-text()`). Avoid `@use '../abstracts/' as *` with bare variable names.

---

## Variables (`abstracts/variables/`)

### Breakpoints

```scss
$breakpoints: (
  'mobile': 0,
  'phablet': 36rem,
  // ~576px
  'tablet': 48rem,
  // ~768px
  'tablet-big': 64rem,
  // ~1024px
  'laptop': 80rem,
  // ~1280px
  'desktop': 100rem, // ~1600px
);
```

### Spacing (base-8 rem scale)

```scss
$spacing: (
  'size-0': 0rem,
  'size-2': 0.125rem,
  'size-4': 0.25rem,
  'size-6': 0.375rem,
  'size-8': 0.5rem,
  'size-12': 0.75rem,
  'size-16': 1rem,
  'size-24': 1.5rem,
  'size-32': 2rem,
  'size-40': 2.5rem,
  'size-48': 3rem,
  'size-64': 4rem,
  'size-80': 5rem,
  'size-128': 8rem,
);
```

Usage: `map.get(variables.$spacing, 'size-16')`

### Colors

Three tiers (see [Colors & Theming](colors-and-theming.md) for full details):

- **Brand colors** (`$pkt-colors`): 46 raw color tokens
- **Semantic colors** (`$pkt-semantic-color-modes`): 104 purpose-mapped tokens as `(light, dark)`
  pairs. `$pkt-semantic-colors` and `$pkt-semantic-colors-dark` are derived from it —
  `token -> colour` per mode — and are what consumers read
- **Deprecated colors** (`$colors`): Legacy tokens — do not use in new code

### Typography

- **Font family**: `$font-family` (Oslo Sans with fallbacks)
- **Font sizes**: `$font-size` map (`size-12` through `size-70`)
- **Font weights**: `$font-weight` map (light 300, regular 400, medium 500, bold 700)
- **Line heights**: `$line-height` map (`lh-20` through `lh-98`)
- **Text styles**: `$pkt-styles` map — 48 predefined combinations (see [Typography](typography.md))

### Other

```scss
$border-width: (
  'thin': 1px,
);
$border-radius: (
  'small': 0.25rem,
  'medium': 0.5rem,
  'large': 1rem,
);
$box-shadow: (
  'small': ...,
  'medium': ...,
  'large': ...,
);
$icon-path: 'https://punkt-cdn.oslo.kommune.no/latest/icons';
```

---

## Mixins (`abstracts/mixins/`)

### `bp($name)` — Breakpoints

```scss
// Named breakpoint
@include bp('tablet-up') { ... }

// Range
@include bp('mobile-to-tablet') { ... }

// Exact breakpoint
@include bp('tablet') { ... }
```

See [Breakpoints & Responsive](breakpoints.md).

### `bp-up($value)` — Breakpoints from a raw value

```scss
// For loops over variables.$breakpoints, where `mobile` is 0
@each $bp-name, $bp-value in variables.$breakpoints {
  @include bp-up($bp-value) { ... }
}
```

Emits no `min-width` when `$value` is `0`. Use this instead of passing a value to `bp()` —
`bp(0)` produces an always-true `@media (min-width: 0)`. See
[Breakpoints & Responsive](breakpoints.md).

### `cq($container-name, $min-width)` — Container Queries

```scss
@include cq('my-container', 48rem) {
  // Applied when container >= 48rem
  // Falls back to media query if container queries unsupported
}
```

### `get-text($name)` — Typography (with line-height)

```scss
@include typography.get-text('pkt-txt-18-medium');
// Outputs: font-size, font-weight, letter-spacing, line-height
```

### `get-text-style($name)` — Typography (without line-height)

```scss
@include typography.get-text-style('pkt-txt-18-medium');
// Outputs: font-size, font-weight, letter-spacing (no line-height)
```

### `truncate-text()` — Text overflow

```scss
@include typography.truncate-text();
// Outputs: overflow: hidden; text-overflow: ellipsis; white-space: nowrap;
```

---

## Functions (`abstracts/functions/`)

| Function                                  | Purpose                                                                              |
| ----------------------------------------- | ------------------------------------------------------------------------------------ |
| `color-token($name)`                      | **Preferred way to use a colour in SCSS.** Returns `var(--pkt-color-$name, <value>)` |
| `map-deep-get($map, $keys...)`            | Access nested Sass map values                                                        |
| `str-replace($string, $search, $replace)` | String replacement (for SVG encoding)                                                |
| `escape-svg($string)`                     | Escape SVG markup for use in `url()` data URIs                                       |

---

## Placeholders (`abstracts/placeholders/`)

```scss
%sr-only   // Screen-reader-only content (visually hidden, accessible)
```

Usage: `@extend %sr-only;`

### Extend placeholders, not classes

`@extend` appends the extending selector to **every rule** the extended selector appears in. On
a placeholder that is cheap. On a heavily used class it is not.

```scss
.pkt-textinput__input {
  @extend .pkt-input;
} // don't
```

`.pkt-input` has 124 rules, so that one line added `.pkt-textinput__input` to all 124 selector
lists. Two such extends cost **6 183 bytes** in `pkt.css` — and both turned out to be dead,
because the components already put `pkt-input` on the field directly. Both files were deleted.

If a component needs the field styling, apply the `pkt-input` class in the markup. If several
components need to share declarations, use a mixin.
