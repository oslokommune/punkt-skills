# Grid System

## Container: `.pkt-container`

A centered wrapper that constrains content width and adds horizontal padding.

### Defaults

- **Padding:** `padding-inline` 16px (1rem) on mobile, 32px (2rem) from tablet up
- **Max-width:** 80rem (1280px) from laptop up
- **Centering:** `margin-inline: auto`

### Width modifiers

| Modifier       | Max-width       | Activates at    |
| -------------- | --------------- | --------------- |
| `--full`       | 100%            | Always          |
| `--phablet`    | 36rem (576px)   | 36rem           |
| `--tablet`     | 48rem (768px)   | 48rem           |
| `--tablet-big` | 64rem (1024px)  | 64rem           |
| `--laptop`     | 80rem (1280px)  | 80rem _(default)_ |
| `--desktop`    | 100rem (1600px) | 100rem          |

Below the activation width, the container is always 100% width.

These are max-width values, not breakpoints. `--phablet` and `--desktop` still exist even
though the `phablet` and `desktop` breakpoints were removed in Punkt 19: the name is the only
thing they ever shared.

### Alignment modifiers

| Modifier   | Effect                |
| ---------- | --------------------- |
| `--left`   | Left-aligned          |
| `--center` | Centered _(default)_  |
| `--right`  | Right-aligned         |

`--center` uses `margin-inline: auto`. `--left` and `--right` are still implemented with physical `margin-left`/`margin-right`, so they do not flip in right-to-left writing modes.

### Examples

```html
<!-- Default: laptop-width centered -->
<div class="pkt-container">...</div>

<!-- Narrow for article text -->
<div class="pkt-container pkt-container--tablet">...</div>

<!-- Full width -->
<div class="pkt-container pkt-container--full">...</div>
```

---

## Grid: `.pkt-grid`

A 12-column CSS Grid for laying out content in columns.

### Defaults

- **Columns:** `repeat(12, 1fr)`
- **Gap:** 16px (1rem) on mobile, 32px (2rem) from tablet up
- **Centering:** `margin-inline: auto`

### Width modifiers

Same as container: `--full`, `--phablet`, `--tablet`, `--tablet-big`, `--laptop`, `--desktop`.
All six are kept, including the two whose names no longer match a breakpoint.

### Alignment modifiers

Same as container: `--left`, `--center`, `--right`.

### Padding modifier

`--padding` — adds `padding-inline` matching the gap sizes (16px mobile, 32px tablet-up).

### Gap modifiers

Override the default gap using spacing size tokens:

| Modifier          | Effect                  |
| ----------------- | ----------------------- |
| `--gap-{size}`    | Both column and row gap |
| `--colgap-{size}` | Column gap only         |
| `--rowgap-{size}` | Row gap only            |

**Responsive gaps:** `--gap-{size}-{breakpoint}-up`, `--colgap-{size}-{breakpoint}-up`, `--rowgap-{size}-{breakpoint}-up`

```html
<!-- Tight gap -->
<div class="pkt-grid pkt-grid--gap-size-8">...</div>

<!-- No gap on mobile, 32px from tablet -->
<div class="pkt-grid pkt-grid--gap-size-0 pkt-grid--gap-size-32-tablet-up">...</div>
```

---

## Cell: `.pkt-cell`

Grid children that control column spanning and alignment.

### Column spanning

`--span{n}` where n is 2–12. Default is 1 column (no modifier needed).

```html
<div class="pkt-grid">
  <div class="pkt-cell pkt-cell--span6">Half width (6/12)</div>
  <div class="pkt-cell pkt-cell--span6">Half width (6/12)</div>
</div>
```

### Responsive spanning

`--span{n}-{breakpoint}-up` — apply span at a breakpoint and above.

```html
<!-- Full width on mobile, half from tablet, third from laptop -->
<div class="pkt-cell pkt-cell--span12 pkt-cell--span6-tablet-up pkt-cell--span4-laptop-up">
  Responsive cell
</div>
```

### Alignment modifiers

**Horizontal:**

- `--left` — align content left
- `--center` — center content
- `--right` — align content right

**Vertical:**

- `--top` — align to top
- `--center-vertical` — center vertically
- `--bottom` — align to bottom

---

## Common patterns

### Two equal columns (mobile stacked)

```html
<div class="pkt-grid">
  <div class="pkt-cell pkt-cell--span12 pkt-cell--span6-tablet-up">Column 1</div>
  <div class="pkt-cell pkt-cell--span12 pkt-cell--span6-tablet-up">Column 2</div>
</div>
```

### Three-column card grid

```html
<div class="pkt-grid">
  <div class="pkt-cell pkt-cell--span12 pkt-cell--span4-tablet-up">Card 1</div>
  <div class="pkt-cell pkt-cell--span12 pkt-cell--span4-tablet-up">Card 2</div>
  <div class="pkt-cell pkt-cell--span12 pkt-cell--span4-tablet-up">Card 3</div>
</div>
```

### Sidebar + main content

```html
<div class="pkt-grid">
  <nav class="pkt-cell pkt-cell--span12 pkt-cell--span3-laptop-up">Sidebar</nav>
  <main class="pkt-cell pkt-cell--span12 pkt-cell--span9-laptop-up">Content</main>
</div>
```

### Responsive grid widths

How each width modifier behaves across breakpoints:

| Modifier       | < 36rem | 36rem | 48rem | 64rem | 80rem | 100rem |
| -------------- | ------- | ----- | ----- | ----- | ----- | ------ |
| `--phablet`    | 100%    | 36rem | 36rem | 36rem | 36rem | 36rem  |
| `--tablet`     | 100%    | 100%  | 48rem | 48rem | 48rem | 48rem  |
| `--tablet-big` | 100%    | 100%  | 100%  | 64rem | 64rem | 64rem  |
| `--laptop`     | 100%    | 100%  | 100%  | 100%  | 80rem | 80rem  |
| `--desktop`    | 100%    | 100%  | 100%  | 100%  | 100%  | 100rem |
| `--full`       | 100%    | 100%  | 100%  | 100%  | 100%  | 100%   |
