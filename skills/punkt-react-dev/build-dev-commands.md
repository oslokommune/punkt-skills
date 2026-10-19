# Build & Dev Commands

From `packages/react/`:

```bash
npm run dev                 # Vite dev server
npm run build               # tsc + vite build
npm run check-tree-shaking  # After build: fails if PktButton alone pulls in other components
npm run lint                # ESLint
npm run lint:fix            # ESLint with auto-fix
npm run format              # Prettier
npm run test                # Vitest single run
npm run test:watch          # Vitest watch mode
```

## Build output

- `dist/punkt-react.js` is the ESM entry. Every source module is its own file next to it
  (`preserveModules`), so `sideEffects: false` lets bundlers drop what isn't imported, and each
  file keeps its `'use client'` directive.
- Bundled dependencies (date-fns, @date-fns/tz, classnames) land in `dist/node_modules/`. npm
  includes them in the tarball.
- `dist/punkt-react.cjs` is a single CommonJS file for `require()`.
- `main`, `module` and `exports` in `package.json` point at the two entry files. `exports` only
  exposes `.`, so the file layout inside `dist/` is not public API.
