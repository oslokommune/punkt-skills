# Build System

## Build tool

Node.js script (`scripts/build.js`) using **Dart Sass** directly. No bundler (Vite/Webpack/Rollup).

## Build process

1. **Clean** — empties `./dist`
2. **Copy** — copies `src/scss/` and `src/scripts/` to `dist/`
3. **Compile** — for each entry file and each component/element:
   - Compile with `sass.compile()` → regular CSS
   - Compile with `style: 'compressed'` → minified CSS (`.min.css`)

## Output structure

```
dist/
├── css/
│   ├── pkt.css                    # Everything (~586KB)
│   ├── pkt.min.css                # Everything minified (~486KB)
│   ├── pkt-base.css               # Base only
│   ├── pkt-components.css         # Components only
│   ├── pkt-elements.css           # Elements only
│   ├── pkt-normalise.css          # Normalize only
│   ├── components/
│   │   ├── accordion.css          # Individual component
│   │   ├── accordion.min.css
│   │   └── ...
│   └── elements/
│       ├── button.css             # Individual element
│       ├── button.min.css
│       └── ...
└── scss/                          # Source SCSS (for consumer @use)
```

## Commands

From `packages/css/`:

```bash
npm run build        # Compile all SCSS to dist/
npm run dev          # Astro dev server (documentation site)
npm run build-app    # Build documentation site
npm run preview      # Preview documentation build
```

## Consumer usage

Consumers can import the full bundle or individual pieces:

```scss
// Full bundle
@use '@oslokommune/punkt-css/dist/scss/pkt';

// Just base styles
@use '@oslokommune/punkt-css/dist/scss/pkt-base';

// Individual component
@use '@oslokommune/punkt-css/dist/scss/components/accordion';
```

Or use the compiled CSS files directly:
```html
<link rel="stylesheet" href="node_modules/@oslokommune/punkt-css/dist/css/pkt.min.css" />
```
