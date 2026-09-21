# Breakpoints

`phablet` (36rem) and `desktop` (100rem) were removed in Punkt 19. The other four keep their
values. `bp('phablet')`, `bp('phablet-up')` and `bp('desktop-up')` now raise a Sass `@error`
that stops the build. Round up to `tablet`, pass a literal value to `bp-up()`, or add the
breakpoint back to `$breakpoints` yourself. Classes ending in `-desktop-up` have no
replacement at all, and `--mobile-up` classes no longer exist because they duplicated the
base class.

To migrate a codebase that is still on Punkt 18, run
`npx @oslokommune/punkt-migrate@19 . --dry-run` from the project root. The tool takes one
path and scans everything under it, skipping `node_modules` and build output.

## Breakpoint scale

| Name         | Min-width | Pixels  |
| ------------ | --------- | ------- |
| `mobile`     | 0         | 0       |
| `tablet`     | 48rem     | ~768px  |
| `tablet-big` | 64rem     | ~1024px |
| `laptop`     | 80rem     | ~1280px |

`mobile` is the base tier. It generates no `--mobile-up` classes, so the three responsive
tiers are `tablet`, `tablet-big` and `laptop`.

## Usage in CSS classes

Breakpoints are used via the `--{breakpoint}-up` suffix on utility classes. This applies the style at the named breakpoint **and above** (min-width).

### Which classes support responsive suffixes

| Class type      | Example                                      |
| --------------- | -------------------------------------------- |
| Typography      | `.pkt-txt-36--tablet-up`                     |
| Spacing         | `.mb-size-24--laptop-up`                     |
| Grid cell spans | `.pkt-cell--span6-tablet-up`                 |
| Grid gaps       | `.pkt-grid--gap-size-32-tablet-up`           |
| Visibility      | `.pkt-hide-tablet-up`, `.pkt-show-laptop-up` |

### Mobile-first pattern

Set the base (mobile) style without a suffix, then override at larger breakpoints:

```html
<!-- Text: 36px mobile, 54px from tablet up -->
<h1 class="pkt-txt-36 pkt-txt-54--tablet-up">Heading</h1>

<!-- Margin: 16px mobile, 32px from tablet, 64px from laptop -->
<div class="mb-size-16 mb-size-32--tablet-up mb-size-64--laptop-up">...</div>

<!-- Grid: full-width mobile, 2 columns from tablet -->
<div class="pkt-cell pkt-cell--span12 pkt-cell--span6-tablet-up">...</div>
```

## Adding your own breakpoints

> Requires SCSS embedding method.

Punkt generates the responsive utility classes from `$breakpoints`, so adding an entry gives
you the classes for it. Use this to bring back `phablet`, or to add a threshold Punkt never
had. The config **must live in its own file**: Sass runs every `@use` before the rest of a
file, so merging in the same file as `@use 'pkt'` happens after Punkt has already generated
its classes, with no error and no effect.

```scss
// punkt-config.scss
@use 'sass:map';
@use '@oslokommune/punkt-css/dist/scss/abstracts/variables' as v;

v.$breakpoints: map.merge(v.$breakpoints, ('phablet': 36rem));
```

```scss
// main.scss
@use 'punkt-config';
@use '@oslokommune/punkt-css/dist/scss/pkt';
```

`map.merge` keeps Punkt's own entries, so components cannot lose a value they depend on.
`@use ... with` does not work here: Sass refuses to configure a module that is already
loaded, and reading the default loads it. The same pattern works for `$spacing`.

Responsive spacing classes are the exception: a new breakpoint gives you typography, grid and
visibility classes immediately, but spacing only if `pkt-spacing-responsive` is also
imported.

## Custom breakpoints in CSS

There is no CSS-only mechanism for custom breakpoints. The `--{breakpoint}-up` suffixes use the fixed scale above. For custom breakpoints in plain CSS, write your own media queries:

```css
@media (min-width: 48rem) {
  .my-element {
    /* tablet and up */
  }
}
```

## SCSS breakpoint mixin

> Requires SCSS embedding method.

```scss
@use '@oslokommune/punkt-css/dist/scss/abstracts/mixins/breakpoints' as *;
```

### Min-width queries (most common)

```scss
@include bp('tablet-up') {
  /* 768px+ */
}
@include bp('tablet-big-up') {
  /* 1024px+ */
}
@include bp('laptop-up') {
  /* 1280px+ */
}
```

### Exact range queries

```scss
@include bp('mobile') {
  /* 0 – 767px */
}
@include bp('tablet') {
  /* 768 – 1023px */
}
@include bp('tablet-big') {
  /* 1024 – 1279px */
}
```

`laptop` is the top tier, so `bp('laptop')` has no upper bound and is identical to
`bp('laptop-up')`.

### Range queries

These still work but warn at build time, and go in Punkt 20. Write the media query directly
in new code.

```scss
@include bp("mobile-to-phablet") { ... }
@include bp("mobile-to-tablet") { ... }
@include bp("mobile-to-laptop") { ... }
@include bp("phablet-to-tablet-big") { ... }
@include bp("tablet-to-tablet-big") { ... }
@include bp("tablet-to-laptop") { ... }
@include bp("tablet-big-to-laptop") { ... }
```

### Custom value

Pass any rem value for a custom min-width query:

```scss
@include bp(50rem) {
  /* 800px+ */
}
```

### Example

```scss
.my-component {
  padding: 1rem;

  @include bp('tablet-up') {
    padding: 2rem;
  }

  @include bp('laptop-up') {
    padding: 3rem;
    max-width: 60rem;
  }
}
```

## SCSS container query mixin

> Requires SCSS embedding method.

For container queries with a media query fallback:

```scss
@use '@oslokommune/punkt-css/dist/scss/abstracts/mixins/container-queries' as *;

.parent {
  container-type: inline-size;
  container-name: my-container;
}

.child {
  @include cq('my-container', 48rem) {
    /* Styles when container is >= 768px wide */
  }
}
```
