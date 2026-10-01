# Component / Element SCSS Patterns

## File structure template

Every component/element file follows this structure:

```scss
/* Component: pkt-component */

// 1. Imports (namespaced)
@use 'sass:map';
@use '../abstracts/variables';
@use '../abstracts/mixins/breakpoints' as *;
@use '../abstracts/mixins/typography';

// 2. Private variables
$-module-name: 'pkt-component';
$-spacing-16: map.get(variables.$spacing, 'size-16');
$-spacing-24: map.get(variables.$spacing, 'size-24');

// 3. Custom element display rule (if Web Component exists)
pkt-component {
  display: block;
}

// 4. Main component class
.pkt-component {
  // Base styles
  padding: $-spacing-16;
  color: var(--pkt-color-text-body-default);

  // 5. Child elements (BEM)
  &__title {
    @include typography.get-text('pkt-txt-18-medium');
    margin-block-end: $-spacing-16;
  }

  &__content {
    @include typography.get-text('pkt-txt-16');
  }

  // 6. Modifiers (BEM)
  &--compact {
    padding: map.get(variables.$spacing, 'size-8');
  }

  &--large {
    padding: $-spacing-24;
  }

  // 7. Skin modifiers (CSS custom properties)
  &--blue {
    --pkt-color-component-bg: var(--pkt-color-surface-default-light-blue);
    background-color: var(--pkt-color-component-bg);
  }

  &--green {
    --pkt-color-component-bg: var(--pkt-color-surface-strong-green);
    background-color: var(--pkt-color-component-bg);
  }

  // 8. Responsive
  @include bp('tablet-up') {
    padding: $-spacing-24;
  }

  // 9. Dark mode
  [data-mode='dark'] & {
    color: var(--pkt-color-brand-neutrals-white);
  }
}
```

## Import conventions

Always use namespaced imports. The standard import block:

```scss
@use 'sass:map';
@use '../abstracts/variables';
@use '../abstracts/mixins/breakpoints' as *;
@use '../abstracts/mixins/typography';
```

- `sass:map` — for `map.get()` calls
- `variables` — namespaced access: `variables.$spacing`, `variables.$pkt-colors`
- `breakpoints as *` — unnamespaced for cleaner `@include bp()` calls
- `typography` — namespaced: `@include typography.get-text()`

## Private variables

Cache frequently used spacing/color values as private variables at the top of the file. Prefix with `-` to indicate file scope:

```scss
$-spacing-8: map.get(variables.$spacing, 'size-8');
$-spacing-16: map.get(variables.$spacing, 'size-16');
$-spacing-24: map.get(variables.$spacing, 'size-24');
```

## Skin pattern

Use modifier classes with CSS custom property overrides. This is the preferred approach for new components:

```scss
.pkt-component {
  background-color: var(--pkt-color-component-bg, transparent);
  border-color: var(--pkt-color-component-border, var(--pkt-color-border-default));
}

.pkt-component--blue {
  --pkt-color-component-bg: var(--pkt-color-surface-default-light-blue);
  --pkt-color-component-border: var(--pkt-color-brand-blue-1000);
}

.pkt-component--green {
  --pkt-color-component-bg: var(--pkt-color-surface-strong-green);
  --pkt-color-component-border: var(--pkt-color-brand-green-1000);
}
```

## Custom element display rules

If the component has a corresponding web component (`pkt-*`), add a display rule before the class definition:

```scss
pkt-accordion {
  display: block;
}

.pkt-accordion {
  // ...
}
```

## Click area for checkbox and radio

`.pkt-input-check__input` is a flex row of `input` + `label`, and the label is the only clickable element. Without help it only covers its own column, so clicks below the control — the dead strip when the label is taller than the control — hit nothing. The label reaches back over that column with a negative `margin-inline-start` and a matching `padding-inline-start`, sized by `--pkt-input-check-control-size`, which is set on the label per control type via sibling selectors (the tile adds the control's inline `1rem` margin and the `2px` border on all four edges). The control keeps `position: relative; z-index: 1` so its own hover and focus styles still work under the label box.

Do not solve this with an absolutely positioned overlay: it blocks text selection in the label and helptext, and turns a selection drag into a toggle.

### Checkbox and radio sizes

Sizes are custom properties, not selector overrides, so the nearest ancestor wins: `.pkt-input-check--{medium,small,xsmall}` on the control, or `.pkt-inputwrapper--{medium,small,xsmall}` on the wrapper around a group. The properties are `--pkt-input-check-label-font-size`, `-label-line-height`, `-helptext-font-size`, `-helptext-line-height`, `-helptext-spacing` and `-tile-padding-block`. Each `var()` falls back to the medium value, so a checkbox outside any wrapper renders as medium. The label and helptext use `get-text-style` (letter-spacing, weight and size, no line-height) and then set `font-size` and `line-height` from the properties, so only the mixin's static `font-size` is overridden.

The size mixin takes the tile height as a spacing token (`size-56`/`size-48`/`size-40`) and derives `--pkt-input-check-tile-padding-block` as `(height − 24px) / 2` (16/12/8px), measured from the outer edge including the 2px border. Pass heights, not paddings: the design specifies tile heights, and the padding follows from them. The control's vertical margin subtracts the border and the difference to the 24px checkbox box: checkbox `- 2px`, radio `± 0` (20px box), switch `- 3px` (26px box), so all three tiles get the same height with the control centred. The tile label is always medium weight.

Horizontally the tile has side padding `--pkt-input-check-tile-padding-inline` (16/12/12px from the outer edge, passed as a spacing token to the size mixin) on both sides, and a fixed 16px between control and text. The label reaches back to the outer edge with a negative margin and adds the control width, the side padding and 1rem as padding. The radio's layout box is 20px with a 2px `box-shadow` ring drawn outside it, so its margin and the label's reach get 2px more; measure the radio by its ring, not its box. With `labelPosition="left"` the geometry is mirrored and the control sits the side padding from the right edge.

### Vertical alignment of label and control

The label's first line is centred on the control, whatever the size or control type. `.pkt-input-check__input` sets `--pkt-input-check-control-height` (24px, radio 20px, switch 26px via `:has`), and whichever is shorter gets half the difference as top offset: the label as `padding-block-start`, the control as `margin-block-start`, both `max(0px, …)`. Only the first line is centred, so wrapped labels and helptext still read from the top. In a tile the controls are normalised to the 24px box, so the label's block padding is `tile padding + (24px − line-height) / 2`, which keeps the tile heights at 56/48/40.

Only the tile grows the label (`flex-grow: 1`, it has a fixed `22.5rem` (360px) width to fill). Non-tile checkboxes and radios deliberately have **no width rule** — no `min(31rem, 100%)` like the other form fields. Their box is whatever the context gives it, and consumers rely on that; forcing a width would stretch the hit area across the full row in block contexts and break anyone laying options out horizontally.

## Formatting

Prettier config (`packages/css/.prettierrc`):
```json
{
  "bracketSpacing": true,
  "printWidth": 120,
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2,
  "useTabs": false
}
```

No stylelint is configured — rely on Dart Sass compiler errors and Prettier for formatting.
