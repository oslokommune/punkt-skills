# Directory Structure

## Package layout

```
packages/css/
├── src/scss/                # All SCSS source
│   ├── abstracts/           # Variables, mixins, functions, placeholders
│   ├── base/                # Global defaults, typography, colors, spacing utilities
│   ├── components/          # Higher-level component styles (31 files)
│   ├── elements/            # Base element styles (11 files)
│   ├── normalise/           # CSS normalize/reset
│   ├── pkt.scss             # Main entry (everything)
│   ├── pkt.layer.scss       # Main entry wrapped in @layer punkt
│   ├── pkt-base.scss        # Base styles only
│   ├── pkt-components.scss  # Components only
│   ├── pkt-elements.scss    # Elements only
│   └── pkt-normalise.scss   # Normalize only
├── dist/                    # Build output
│   ├── css/                 # Compiled CSS (regular + .min.css)
│   │   ├── components/      # Individual component CSS
│   │   └── elements/        # Individual element CSS
│   └── scss/                # Source SCSS (copied for consumers)
├── scripts/                 # Build scripts (Node.js)
├── package.json
└── .prettierrc
```

## Components vs Elements

- **Elements** (`elements/`): Styles for basic HTML-level controls — button, checkbox, radio, input, select, table, form, list, image, hr, section.
- **Components** (`components/`): Styles for higher-level UI patterns — accordion, alert, badge, card, modal, tabs, header, footer, etc.

## Entry points

| File | Contents |
|---|---|
| `pkt.scss` | Everything (normalise + base + elements + components) |
| `pkt.layer.scss` | Everything, wrapped in `@layer punkt` |
| `pkt-base.scss` | Base styles only |
| `pkt-elements.scss` | Elements only |
| `pkt-components.scss` | Components only |
| `pkt-normalise.scss` | Normalize/reset only |
| `pkt-tokens.scss` | Color variables, dark mode, `@font-face` |
| `pkt-utilities.scss` | Spacing, color and visibility helper classes. Requires `pkt-tokens` |
| `pkt-grid.scss` | Containers, grid and layouts. Requires `pkt-tokens` |
| `pkt-docs.scss` | Everything plus docs-only styles. Internal to the documentation site |

`pkt-tokens`, `pkt-utilities` and `pkt-grid` are the granular entries, so a consumer can take the design tokens and grid without pulling in every component style. Both `pkt-utilities` and `pkt-grid` depend on the custom properties in `pkt-tokens` — load it first or they resolve against nothing.

Each entry file uses `@forward` to include its modules:
```scss
// pkt.scss
@charset "utf-8";
@forward 'normalise';
@forward 'base';
@forward 'elements';
@forward 'components';
```

`pkt.layer.scss` is the exception. It has to use `meta.load-css`, because Sass does not allow `@use` inside `@layer`:

```scss
// pkt.layer.scss
@charset "utf-8";
@use 'sass:meta';

@layer punkt {
  @include meta.load-css('pkt');
}
```

Keep all of Punkt inside the single `punkt` layer. Splitting it across layers would let Punkt's own styles compete with each other — base link styles could start beating component button styles.

## Index files

Every directory has an `_index.scss` that `@forward`s all its partials. When adding a new component or element, register it in the corresponding `_index.scss`.

`components/_index.scss` and `elements/_index.scss` are alphabetical — add new entries in order.

**`base/_index.scss` is not.** Its order is deliberate and mirrors the granular entry points:

```
tokens      oslo-sans, colors-tokens
reset       defaults
typography  typography, link
grid        containers, grid, layouts
utilities   colors-utilities, spacing, visibility
a11y        accessibility
```

Do not sort it. **Utilities come last on purpose.** They currently win through `!important`, but
the plan is to remove those — and then source order is the only thing left making a utility
class beat a component style. With utilities last that still works; sorted alphabetically it
would not, and the breakage would be silent.

`base/_link.scss` holds the whole `.pkt-link` component — `a`, `.pkt-link-button`, their states,
`.pkt-link__icon`, `--icon-left/--icon-right`, `--external` and the dark-mode filters. It lives in
`base/` rather than `components/` because the `a` element rules must stay ahead of
`elements/_button.scss`, so `<a class="pkt-btn">` still gets button styling.

`base/_spacing-responsive.scss` and `base/_grid-responsive.scss` hold the per-breakpoint variants
of the spacing and grid utilities — the bulk of the classes in both. They are separate files so
they can be dropped from `pkt-base` by removing a single `@forward` from `base/_index.scss`, where
both are currently forwarded. Put new responsive utility loops in these files rather than beside
the non-responsive ones.

`base/_defaults.scss` holds `.pkt-contents` (`display: contents`). It is not a utility — around
eleven components emit it as a slot wrapper, so it has to load whenever components do. `pkt-base`
is mandatory, which is what makes `base/` the right home.

`base/_colors.scss` is a forwarding stub over `_colors-tokens.scss` (the `:root` custom
properties and dark mode) and `_colors-utilities.scss` (the `.pkt-color-*` helper classes). The
split is what lets `pkt-tokens` ship the variables without the classes. Add new colour tokens to
`_colors-tokens.scss`, new helper classes to `_colors-utilities.scss`.
