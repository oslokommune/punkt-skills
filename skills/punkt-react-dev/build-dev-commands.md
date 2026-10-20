# Build & Dev Commands

From `packages/react/`:

```bash
npm run dev                 # Vite dev server
npm run build               # tsc + vite build
npm run check-tree-shaking  # After build: fails if PktButton alone pulls in other components
npm run check-browser-support  # After build: fails on syntax or APIs that Safari 15 can't run
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
- `build.target` is Vite's default except Safari and iOS, which are lowered to 15, so the output
  runs on older iPhones. Vite 8's own default is Safari 16.4. `check-browser-support` runs in the
  PR workflow and fails if anything newer slips through.
- `main`, `module` and `exports` in `package.json` point at the two entry files. `exports` only
  exposes `.`, so the file layout inside `dist/` is not public API.
