# Dev Pages

## Overview

The React package includes a **Vite + React Router** dev app for manually testing and demonstrating components. Each component has a dedicated route page in `src/routes/`.

## Architecture

- **Entry point:** `src/main.tsx` — creates a `BrowserRouter` with all component routes
- **Layout wrapper:** `src/routes/root.tsx` — renders a dev header with navigation dropdown, and a `.dev-layout` grid with the `<Outlet />` and the table of contents
- **Dev header:** `src/dev-components/DevHeader.tsx` — logo + select dropdown for navigating between components
- **Table of contents:** `src/dev-components/DevTableOfContents.tsx` — lists the page's first `<h1>` (as title), `<h2>` and `<h3>` (indented). Smooth-scrolls on click and highlights the section in view. Sticky on the right from `laptop-up`, a sticky menu bar (current section + dropdown) on smaller screens. Hidden when the page has fewer than two `<h2>`/`<h3>`
- **Styles:** `src/index.scss` — dev-specific layout styles (`.page-container`, `.dev-layout`, `.dev-toc`, `.page-main`, `.component`)

## Route file convention

Each component gets a route file at `src/routes/{name}.tsx`:

```typescript
import AlertSpec from 'componentSpecs/alert.json'
import { useState } from 'react'

import { IPktAlert } from '@/components/alert/Alert'
import IAlertSpec from '@/interfaces/IAlertSpec'
import { PktAlert } from '..'

export default function Alert() {
  const alertSpec = AlertSpec as unknown as IAlertSpec
  const [specProps] = useState(alertSpec.props)

  return (
    <main className="page-main pkt-container">
      <h1>Alert</h1>
      {specProps.skin.type.map((skinType: IPktAlert['skin'], index: number) => (
        <section className="component" key={`skin-${index}`}>
          <h2>Skin: {skinType}</h2>
          <PktAlert skin={skinType}>Content here</PktAlert>
        </section>
      ))}
    </main>
  )
}
```

### Key patterns

- **Default export** — each route file exports a single default function component
- **`page-main` wrapper** — use `<main className="page-main pkt-container">` as the outer element
- **`component` sections** — wrap each demo variant in `<section className="component">` for consistent card styling
- **Component specs** — import from `componentSpecs/{name}.json` to iterate over prop variants (skins, sizes, states)
- **Interactive demos** — add event handlers, form submissions, state toggles to test behavior
- **Headings** — every `<h1>`, `<h2>` and `<h3>` gets an `id` automatically (slug of the text, via `ensureHeadingIds` in `shared-utils/heading-ids`) and appears in the table of contents (only the first `<h1>`). Set an `id` yourself only if you need a stable anchor
- **`hideHeader`** — for full-page layouts (e.g. header-full; also hides the table of contents), add `hideHeader: true` to the route entry in `root.tsx`

## Registering a new dev page

Three files need to be updated:

### 1. Create the route file

Create `src/routes/{name}.tsx` with the component demo.

### 2. Add to the router in `src/main.tsx`

```typescript
import NewComponent from './routes/newcomponent'

// In the children array:
{ path: '/newcomponent', element: <NewComponent /> },
```

### 3. Add to the navigation in `src/routes/root.tsx`

```typescript
// In the routes array:
{ path: '/newcomponent', label: 'NewComponent' },
```

## Commands

```bash
npm run dev          # Start Vite dev server with HMR
npm run build-app    # Build the dev app (output: dist-app/)
npm run preview      # Preview built dev app
```
