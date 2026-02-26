# Breakpoints & Responsive

## Breakpoint values

| Name | Value | Pixels |
|---|---|---|
| `mobile` | 0 | 0px |
| `phablet` | 36rem | ~576px |
| `tablet` | 48rem | ~768px |
| `tablet-big` | 64rem | ~1024px |
| `laptop` | 80rem | ~1280px |
| `desktop` | 100rem | ~1600px |

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
