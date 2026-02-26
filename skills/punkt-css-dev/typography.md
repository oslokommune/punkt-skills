# Typography

## Font

- **Family**: Oslo Sans (loaded from `@oslokommune/punkt-assets`)
- **Weights**: Light (300), Regular (400), Medium (500), Bold (700)
- **Icon font**: Oslo Icons
- **Loading**: `font-display: swap` for performance

## Base defaults

```scss
body {
  font-family: $font-family;     // Oslo Sans + fallbacks
  font-size: 1rem;               // 16px
  font-weight: 300;              // Light
  line-height: 1.75rem;          // 28px
  -webkit-font-smoothing: antialiased;
}
```

All headings (`h1`–`h6`) are reset to `font-size: 1rem; font-weight: normal`. Use text utility classes or the `get-text` mixin to style them.

## Text style system

48 predefined text styles in the `$pkt-styles` map. Each defines size, weight, letter-spacing, and line-height.

**Pattern**: `pkt-txt-{size}[-{weight}]`

**Available sizes**: 12, 14, 16, 18, 20, 22, 24, 26, 28, 30, 36, 40, 48, 54, 70

**Weight variants**: `-light`, (default = regular), `-medium`, `-bold`

**Examples**: `pkt-txt-16`, `pkt-txt-18-medium`, `pkt-txt-24-bold`, `pkt-txt-54-light`

## Using in components

### Mixin (preferred in component SCSS)

```scss
@use '../abstracts/mixins/typography';

.pkt-component__title {
  @include typography.get-text('pkt-txt-18-medium');
  // Outputs: font-size, font-weight, letter-spacing, line-height
}

.pkt-component__subtitle {
  @include typography.get-text-style('pkt-txt-14');
  // Same but without line-height (useful when line-height is set elsewhere)
}
```

### Utility classes (in HTML)

```html
<h2 class="pkt-txt-24-bold">Heading</h2>
<p class="pkt-txt-16">Body text</p>
```

### Responsive variants

```html
<h2 class="pkt-txt-18-bold pkt-txt-24-bold--tablet-up pkt-txt-36-bold--laptop-up">
  Responsive heading
</h2>
```

## Other typography utilities

| Class | Purpose |
|---|---|
| `.pkt-txt-start` | `text-align: start` |
| `.pkt-txt-center` | `text-align: center` |
| `.pkt-txt-end` | `text-align: end` |
| `.pkt-truncate-text` | Single-line truncation with ellipsis |
| `.pkt-font-light` | `font-weight: 300` |
| `.pkt-font-regular` | `font-weight: 400` |
| `.pkt-font-medium` | `font-weight: 500` |
| `.pkt-font-bold` | `font-weight: 700` |
| `.pkt-display-contents` | `display: contents` |
