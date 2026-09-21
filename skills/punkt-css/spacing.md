# Spacing

## Utility class format

`{type}{direction}-{size}` with optional `--{breakpoint}-up` for responsive spacing.

### Type

| Prefix | Property |
| ------ | -------- |
| `m`    | margin   |
| `p`    | padding  |

There is no `gap` utility. The standalone `gap-size-*` classes were removed in Punkt 19 with
no replacement: set `gap` in your own CSS, or use the grid class `pkt-grid--gap-size-16`.

### Direction

The suffixes are named after physical directions, but they all resolve to **logical** CSS properties, so spacing follows the writing direction automatically.

| Suffix   | Property              | Sides in LTR   |
| -------- | --------------------- | -------------- |
| `t`      | `margin-block-start`  | top            |
| `r`      | `margin-inline-end`   | right          |
| `b`      | `margin-block-end`    | bottom         |
| `l`      | `margin-inline-start` | left           |
| `x`      | `margin-inline`       | left + right   |
| `y`      | `margin-block`        | top + bottom   |
| _(none)_ | `margin`              | all sides      |

Padding utilities map the same way (`.pt-` → `padding-block-start`, and so on).

### The scale

These 19 values are the approved scale and mirror Figma exactly. Use these in new code.

| Token      | Value    | Pixels |
| ---------- | -------- | ------ |
| `size-0`   | 0        | 0      |
| `size-2`   | 0.125rem | 2px    |
| `size-4`   | 0.25rem  | 4px    |
| `size-6`   | 0.375rem | 6px    |
| `size-8`   | 0.5rem   | 8px    |
| `size-12`  | 0.75rem  | 12px   |
| `size-16`  | 1rem     | 16px   |
| `size-24`  | 1.5rem   | 24px   |
| `size-32`  | 2rem     | 32px   |
| `size-40`  | 2.5rem   | 40px   |
| `size-48`  | 3rem     | 48px   |
| `size-56`  | 3.5rem   | 56px   |
| `size-64`  | 4rem     | 64px   |
| `size-72`  | 4.5rem   | 72px   |
| `size-80`  | 5rem     | 80px   |
| `size-88`  | 5.5rem   | 88px   |
| `size-96`  | 6rem     | 96px   |
| `size-104` | 6.5rem   | 104px  |
| `size-128` | 8rem     | 128px  |

### Half-steps — supported, but undocumented

These sit between the Figma steps. They are not part of the approved scale and are not shown in
the documentation, but they are in heavy use across several teams and will stay. Use the nearest
scale value in new code.

`size-10` (10px), `size-20` (20px)

### Removed in Punkt 19

These were never in Figma and had almost no usage. They no longer exist. A `map.get` for one
of them returns `null`, which makes the declaration vanish without a build error.

| Token      | Pixels | Replace with |
| ---------- | ------ | ------------ |
| `size-5`   | 5px    | `size-6`     |
| `size-15`  | 15px   | `size-16`    |
| `size-30`  | 30px   | `size-32`    |
| `size-50`  | 50px   | `size-48`    |
| `size-52`  | 52px   | `size-56`    |
| `size-60`  | 60px   | `size-64`    |
| `size-75`  | 75px   | `size-72`    |
| `size-100` | 100px  | `size-104`   |

To migrate a codebase that is still on Punkt 18, run
`npx @oslokommune/punkt-migrate@19 . --dry-run` from the project root. The tool takes one path
and scans everything under it, skipping `node_modules` and build output.

## Examples

```html
<!-- Margin bottom 24px -->
<p class="mb-size-24">Paragraph with bottom margin</p>

<!-- Padding on all sides 16px -->
<div class="p-size-16">Padded box</div>

<!-- Horizontal padding 12px -->
<div class="px-size-12">Left and right padding</div>

<!-- Vertical margin 32px -->
<section class="my-size-32">Section with top and bottom margin</section>

<!-- Reset margin right to 0 -->
<div class="mr-size-0">No right margin</div>

```

## Responsive spacing

These classes live in a separate file that is **not** included by `pkt` or `pkt-base`. Import
`pkt-spacing-responsive` (or `pkt-spacing-responsive.layer` when using `@layer`) to get them:

```scss
@use '@oslokommune/punkt-css/dist/scss/pkt';
@use '@oslokommune/punkt-css/dist/scss/pkt-spacing-responsive';
```

Add `--{breakpoint}-up` to apply spacing at a breakpoint and above:

```html
<!-- 16px bottom margin on mobile, 32px from tablet up -->
<p class="mb-size-16 mb-size-32--tablet-up">Responsive margin</p>

<!-- 12px horizontal padding on mobile, 24px from laptop up -->
<div class="px-size-12 px-size-24--laptop-up">Responsive padding</div>

<!-- No padding on mobile, 32px from tablet up -->
<div class="p-size-0 p-size-32--tablet-up">Progressive padding</div>
```

## SCSS usage

> Requires SCSS embedding method.

```scss
@use 'sass:map';
@use '@oslokommune/punkt-css/dist/scss/abstracts/variables';

.my-element {
  padding: map.get(variables.$spacing, 'size-24');
  margin-block-end: map.get(variables.$spacing, 'size-16');
}
```

## Adding your own values

Punkt generates the utility classes from `$spacing`, so adding an entry gives you the classes
for it. Use this to bring back a removed value, or to add one Punkt never had. The config
**must live in its own file**, for the same reason as for breakpoints: Sass runs every `@use`
before the rest of a file.

```scss
// punkt-config.scss
@use 'sass:map';
@use '@oslokommune/punkt-css/dist/scss/abstracts/variables' as v;

v.$spacing: map.merge(v.$spacing, ('size-52': 3.25rem, 'size-200': 12.5rem));
```

```scss
// main.scss
@use 'punkt-config';
@use '@oslokommune/punkt-css/dist/scss/pkt';
```

`map.merge` keeps Punkt's own entries, so components cannot lose a value they depend on. The
order of `$spacing` decides which class wins when two of the same type sit on one element.
