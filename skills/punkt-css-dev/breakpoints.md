# Breakpoints & Responsive

## Breakpoint values

| Name | Value | Pixels |
|---|---|---|
| `mobile` | 0 | 0px |
| `tablet` | 48rem | ~768px |
| `tablet-big` | 64rem | ~1024px |
| `laptop` | 80rem | ~1280px |

`phablet` (36rem) and `desktop` (100rem) were removed in Punkt 19. `bp('phablet')`,
`bp('phablet-up')` and `bp('desktop-up')` raise an `@error`. Use `bp-up()` with a literal
value when a component needs a threshold that is not in the map. Generator loops must skip
the tier whose value is `0`, or they emit `--mobile-up` duplicates of the base class.

## The `bp()` mixin

**Always use the `bp()` mixin** for responsive styles. Do not write raw `@media` queries.

```scss
@use '../abstracts/mixins/breakpoints' as *;
```

### Up variants (min-width)
```scss
@include bp('tablet-up') {
  // Applies at tablet and above (>= 48rem)
}

@include bp('laptop-up') {
  // Applies at laptop and above (>= 80rem)
}
```

### Exact breakpoint
```scss
@include bp('tablet') {
  // Applies only at tablet range (48rem to 63.99rem)
}
```

### Range variants
```scss
@include bp('mobile-to-tablet') {
  // Applies from mobile to just below tablet (0 to 47.99rem)
}

@include bp('tablet-to-laptop') {
  // Applies from tablet to just below laptop (48rem to 79.99rem)
}
```

## The `bp-up()` mixin — for loops over `$breakpoints`

`bp()` takes a **name**. `bp-up()` takes a **raw value** and is what you want when generating
classes in a loop:

```scss
@mixin bp-up($value)   // @media screen and (min-width: $value), or plain @media screen when $value is 0
```

Use it whenever you iterate `variables.$breakpoints`, because `mobile` is `0`:

```scss
@each $bp-name, $bp-value in variables.$breakpoints {
  @include bp-up($bp-value) { ... }
}
```

**Do not pass a value to `bp()`.** `bp('#{$bp-value}')` falls through to the generic `@else`
branch and emits `@media screen and (min-width: 0)` for mobile — a query that is always true.
`bp-up()` drops the `min-width` entirely when the value is `0`.

## Generating responsive classes: loop breakpoint-outermost

When a utility set is generated per (value × breakpoint), put the **breakpoint loop outside**
and the `bp-up()` include **outside the value loop**:

```scss
// Right — one @media block per breakpoint
@each $bp-name, $bp-value in variables.$breakpoints {
  @include bp-up($bp-value) {
    @each $name, $value in $some-map {
      .u-#{$name}--#{$bp-name}-up { prop: $value; }
    }
  }
}
```

```scss
// Wrong — one @media block per class
@each $name, $value in $some-map {
  @each $bp-name, $bp-value in variables.$breakpoints {
    @include bp-up($bp-value) { .u-#{$name}--#{$bp-name}-up { prop: $value; } }
  }
}
```

The wrong form orders output by value rather than by breakpoint, so with equal specificity the
*widest* declared value wins instead of the narrowest breakpoint — `mb-size-32--tablet-up
mb-size-16--laptop-up` gives 32px at laptop. It also emits one `@media` block per class instead
of one per breakpoint, which scatters near-identical rules across the file.

**Don't go further and merge the remaining `@media` blocks.** Block count is not a size target:
the blocks are separated by ordinary rules, so merging them means reordering, and moving rules
away from the near-identical text they compress against makes the gzipped file *larger* even
though the raw file shrinks. Since assets are served gzipped, that is the number that counts.

## When to use which

- **Container queries** are preferred for **component** development — components should respond to their own available space, not the viewport.
- **Media queries** (`bp()` mixin) are preferred for **layouts** — page-level structure, grid configuration, and utility classes that depend on viewport size.

## Container queries

For component-level responsive behavior, use the `cq()` mixin which provides a media query fallback:

```scss
@use '../abstracts/mixins/container-queries' as *;

.pkt-component__wrapper {
  container: component / inline-size;
}

@include cq('component', 48rem) {
  .pkt-component__content {
    flex-direction: row;
  }
}
```

Native `@container` is also used directly in newer components:

```scss
.pkt-input__range-inputs {
  container: range / inline-size;
}

@container range (width < 30rem) {
  .pkt-input__container {
    grid-template-columns: min-content auto 4rem;
  }
}
```

## Approach

Mobile-first: write base styles for mobile, then add overrides with `-up` breakpoints.
