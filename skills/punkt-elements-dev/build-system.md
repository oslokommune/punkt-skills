# Build System

## Build tool

**Vite** with `vite-plugin-dts` for TypeScript declarations and `vite-plugin-web-components-hmr` for Hot Module Replacement in dev mode.

## Library build (`vite.config.ts`)

Builds the component library for npm distribution:

- **Format:** ES modules (`.js`) + CommonJS (`.cjs`)
- **Entries:** Separate entry for each component + a combined `index` entry
- **Naming:** `pkt-{name}.js` / `pkt-{name}.cjs`
- **Types:** `.d.ts` files via `vite-plugin-dts` with `rollupTypes: true`
- **Externals:** `lit` is not bundled (peer dependency)

### Output structure

```
dist/
├── pkt-index.js              # Combined entry (all components)
├── pkt-button.js             # Individual component (ES)
├── pkt-button.cjs            # Individual component (CJS)
├── button.d.ts               # TypeScript declarations
├── pkt-select.js
├── pkt-select.cjs
├── select.d.ts
└── ...
```

### Path aliases

The Vite config maps the same aliases as `tsconfig.json`:

```typescript
resolve: {
  alias: {
    'shared-types': '../../shared-types/',
    'shared-utils': '../../shared-utils/',
    'shared-strings': '../../shared-strings/',
    '@': path.resolve(__dirname, 'src'),
    'pkt': '@oslokommune/punkt-css/dist',
    'pktAssets': '@oslokommune/punkt-assets/dist',
    'componentSpecs': '../../component-specs/',
  }
}
```

## Dev app build (`vite.config-app.ts`)

Builds the documentation/demo app from `src/docs/` and `index.html`. Output goes to `dist-app/`.

## Commands

From `packages/elements/`:

```bash
npm run dev          # Vite dev server with HMR
npm run build        # tsc + vite build (library)
npm run build-app    # tsc + vite build (documentation app)
npm run preview      # Preview built documentation app
npm run type-check   # tsc --noEmit (type checking only)
```

## Consumer usage

```javascript
// Import individual components (tree-shakable)
import '@oslokommune/punkt-elements/dist/pkt-button.js'
import '@oslokommune/punkt-elements/dist/pkt-select.js'

// Import all components
import '@oslokommune/punkt-elements'
```

## Adding a new component to the build

Nothing to do. `vite.config.ts` reads `src/components/` and creates one entry per directory, plus `index`:

```typescript
const components = fs
  .readdirSync(baseFolder)
  .filter((f) => fs.statSync(path.join(baseFolder, f)).isDirectory())

const entries = {
  index: baseFolder + 'index.ts',
  ...components.reduce((obj, name) => ({ ...obj, [name]: baseFolder + name }), {}),
}
```

Every directory in `src/components/` therefore needs an `index.ts`, or the build fails.
