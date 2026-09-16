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

`.pkt-input-check__input` is a flex row of `input` + `label`, and the label is the only clickable element. Without help it only covers its own column, so clicks below the control — the dead strip when the label is taller than the control — hit nothing. The label reaches back over that column with a negative `margin-inline-start` and a matching `padding-inline-start`, sized by `--pkt-input-check-control-size`, which is set on the label per control type via sibling selectors (the tile adds the control's `1rem` margin and the `2px` border on all four edges). The control keeps `position: relative; z-index: 1` so its own hover and focus styles still work under the label box.

Do not solve this with an absolutely positioned overlay: it blocks text selection in the label and helptext, and turns a selection drag into a toggle.

Only the tile grows the label (`flex-grow: 1`, it has a fixed `21.875rem` width to fill). Non-tile checkboxes and radios deliberately have **no width rule** — no `min(31rem, 100%)` like the other form fields. Their box is whatever the context gives it, and consumers rely on that; forcing a width would stretch the hit area across the full row in block contexts and break anyone laying options out horizontally.

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
