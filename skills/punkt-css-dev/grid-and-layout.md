# Grid & Layout

## Grid system

12-column CSS Grid with responsive gaps.

### Base grid
```html
<div class="pkt-grid">
  <div class="pkt-cell--span6">Half width</div>
  <div class="pkt-cell--span6">Half width</div>
</div>
```

### Grid width modifiers
- `.pkt-grid--full` — full viewport width
- `.pkt-grid--phablet` — max-width at phablet breakpoint
- `.pkt-grid--tablet` — max-width at tablet breakpoint
- `.pkt-grid--laptop` — max-width at laptop breakpoint (default)
- `.pkt-grid--desktop` — max-width at desktop breakpoint

### Grid alignment
- `.pkt-grid--left`
- `.pkt-grid--center`
- `.pkt-grid--right`

### Gap modifiers
- `.pkt-grid--gap-size-{token}` — both column and row gap
- `.pkt-grid--colgap-size-{token}` — column gap only
- `.pkt-grid--rowgap-size-{token}` — row gap only
- Responsive: `.pkt-grid--gap-size-16--tablet-up`

### Cell spans
- `.pkt-cell--span1` through `.pkt-cell--span12`
- Responsive: `.pkt-cell--span6--tablet-up`, `.pkt-cell--span4--laptop-up`

## Container

Centered container with responsive padding:

```html
<div class="pkt-container">
  <!-- Content -->
</div>
```

### Container width modifiers
- `.pkt-container--full`
- `.pkt-container--phablet`
- `.pkt-container--tablet`
- `.pkt-container--tablet-big`
- `.pkt-container--laptop` (default max-width: 80rem)
- `.pkt-container--desktop`

### Container alignment
- `.pkt-container--left`
- `.pkt-container--center`
- `.pkt-container--right`

## Visibility utilities

| Class | Effect |
|---|---|
| `.pkt-hide` | `display: none` |
| `.pkt-show` | `display: block` |
| `.pkt-hide-{breakpoint}-up` | Hidden from breakpoint and above |
| `.pkt-show-{breakpoint}-up` | Visible from breakpoint and above |
