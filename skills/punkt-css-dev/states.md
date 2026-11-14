# States & Interactivity

## Interactive state pattern

All interactive components provide both pseudo-class states and corresponding BEM modifier classes (for documentation/testing):

```scss
.pkt-btn {
  // Normal
  color: var(--pkt-color-button-text-normal);
  background: var(--pkt-color-button-background-normal);

  // Hover
  &:hover,
  &.pkt-btn--hover {
    color: var(--pkt-color-button-text-hover);
    text-decoration: underline;
  }

  // Focus (keyboard only)
  &:focus-visible,
  &.pkt-btn--focus {
    box-shadow:
      0px 0px 0px 0.125rem var(--pkt-color-text-action-active),
      0px 0px 0px 0.375rem var(--pkt-color-border-states-focus);
    outline: none;
  }

  // Active
  &:active,
  &.pkt-btn--active {
    color: var(--pkt-color-button-text-active);
  }

  // Disabled
  &:disabled,
  &[disabled],
  &[data-disabled] {
    color: var(--pkt-color-button-text-disabled);
    cursor: inherit;
    pointer-events: none;
  }
}
```

## Forced state classes

Every state pseudo-class has a forced modifier class in the same selector list, so docs, demos
and screenshots can show the state without a mouse or keyboard:

| Pseudo-class                                | Forced class |
| ------------------------------------------- | ------------ |
| `:hover`                                    | `--hover`    |
| `:focus`, `:focus-visible`, `:focus-within` | `--focus`    |
| `:active`                                   | `--active`   |
| `:visited`                                  | `--visited`  |

The class is built from the element class of the compound that carries the pseudo-class, with
any modifier stripped: `&--error[type='checkbox']:hover` on `.pkt-input-check__input-checkbox`
gets `&--error[type='checkbox'].pkt-input-check__input-checkbox--hover`. A bare `a` uses the
`pkt-link--*` classes.

```scss
&:hover > .pkt-input-check__input-checkbox + .pkt-input-check__input-label,
&.pkt-input-check__input--hover > .pkt-input-check__input-checkbox + .pkt-input-check__input-label {
  …
}
```

Rules for writing them:

- Add the forced selector next to the pseudo one, never in a separate rule, so the two keep the
  same declarations.
- A negated state keeps its specificity with `:where()`:
  `&:focus:not(:active), &.pkt-x--focus:not(:active):where(:not(.pkt-x--active))`. A plain
  `:not(.pkt-x--active)` adds a class to the specificity and can beat later rules.
- Don't put `:has()` in a selector list together with the pseudo selector. Browsers without
  `:has()` (old iPads) drop the whole list, real state included.
- Skip rules that only reset (`:focus { outline: none }`); they have no visible state to force.

How to use them: put the class on every element that would match the pseudo-class. Real focus
also makes each ancestor match `:focus-within`, so a container with a `:focus-within` rule needs
its own `--focus` (`pkt-input__container--focus` next to `pkt-input--focus`). An element with two
blocks needs both (`pkt-input-icon--hover pkt-btn--hover`, `pkt-card__link--focus pkt-link--focus`).

## Focus ring technique

The standard focus ring uses a **double box-shadow** (inner ring + outer ring):

```scss
&:focus-visible {
  box-shadow:
    0px 0px 0px 0.125rem var(--pkt-color-text-action-active),
    // Inner ring (2px)
    0px 0px 0px 0.375rem var(--pkt-color-border-states-focus); // Outer ring (6px)
  outline: none;
}
```

Links don't style `:focus:not(:focus-visible)`. A clicked link keeps focus, so an outline there
stays visible after the click, and it showed up all over the header.

## Disabled state

Always handle all three disabled variants:

```scss
&:disabled,
&[disabled],
&[data-disabled] {
  // Muted colors
  cursor: inherit;
  pointer-events: none;
}
```

- `:disabled` — native HTML disabled attribute (form elements)
- `[disabled]` — attribute selector (works on any element)
- `[data-disabled]` — data attribute (used by web components)

---

## Avoid naming other components in `:not()`

`:not(:focus-visible)` and similar — narrowing on the element's own state — is fine. Naming
_other components_ to carve yourself out around them is not:

```scss
// _input.scss, existing — don't add more of these
&--small:not(:is(textarea, .pkt-timepicker__input, .pkt-datepicker__input)) { … }
&__container:is(:has(.pkt-datepicker__input:not(.pkt-datepicker--multiple))) { … }
```

The rule is generic, so it has to enumerate its own exceptions. That makes `_input.scss` know
about every component built on an input: add a new input-based component and you must edit
`_input.scss` too. The same pattern sits in `base/_typography.scss`, where the visited-link
colour excludes `.pkt-linkcard`, `.pkt-btn` and `.pkt-tabs__link`.

It is a knowledge coupling, not an import, so it survives any amount of moving files around.

Prefer the inverse: make the generic rule opt-in via a modifier class the component applies, so
the general case does not need a list of special cases.
