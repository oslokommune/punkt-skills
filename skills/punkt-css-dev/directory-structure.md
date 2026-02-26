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
| `pkt-base.scss` | Base styles only |
| `pkt-elements.scss` | Elements only |
| `pkt-components.scss` | Components only |
| `pkt-normalise.scss` | Normalize/reset only |

Each entry file uses `@forward` to include its modules:
```scss
// pkt.scss
@charset "utf-8";
@forward 'normalise';
@forward 'base';
@forward 'elements';
@forward 'components';
```

## Index files

Every directory has an `_index.scss` that `@forward`s all its partials. When adding a new component or element, register it in the corresponding `_index.scss`.
