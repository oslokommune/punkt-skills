# Spacing

## Token scale

Base-8 system using rem units:

| Token | Value | Pixels |
|---|---|---|
| `size-0` | 0rem | 0px |
| `size-2` | 0.125rem | 2px |
| `size-4` | 0.25rem | 4px |
| `size-6` | 0.375rem | 6px |
| `size-8` | 0.5rem | 8px |
| `size-12` | 0.75rem | 12px |
| `size-16` | 1rem | 16px |
| `size-24` | 1.5rem | 24px |
| `size-32` | 2rem | 32px |
| `size-40` | 2.5rem | 40px |
| `size-48` | 3rem | 48px |
| `size-64` | 4rem | 64px |
| `size-80` | 5rem | 80px |
| `size-128` | 8rem | 128px |

## Usage in component SCSS

Access via the `$spacing` map:

```scss
@use 'sass:map';
@use '../abstracts/variables';

// Cache as private variables at file top
$-spacing-8: map.get(variables.$spacing, 'size-8');
$-spacing-16: map.get(variables.$spacing, 'size-16');
$-spacing-24: map.get(variables.$spacing, 'size-24');

.pkt-component {
  padding: $-spacing-16;
  gap: $-spacing-8;
  margin-bottom: $-spacing-24;
}
```

## Utility classes

All utility classes use `!important` (designed to override component styles).

### Margin
- `.m-size-{token}` — all sides
- `.mt-size-{token}` — top
- `.mb-size-{token}` — bottom
- `.ml-size-{token}` — left
- `.mr-size-{token}` — right
- `.mx-size-{token}` — left + right
- `.my-size-{token}` — top + bottom

### Padding
- `.p-size-{token}` — all sides
- `.pt-size-{token}`, `.pb-size-{token}`, `.pl-size-{token}`, `.pr-size-{token}`
- `.px-size-{token}`, `.py-size-{token}`

### Gap
- `.gap-size-{token}` — flex/grid gap

### Responsive variants
All spacing utilities have responsive suffixes:
- `.m-size-16--tablet-up`, `.p-size-24--laptop-up`, `.gap-size-8--phablet-up`
