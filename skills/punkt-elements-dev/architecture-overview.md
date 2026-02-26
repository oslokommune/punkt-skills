# Architecture Overview

Punkt is a monorepo design system by Oslo kommune. The Elements package provides Lit-based web components that consume BEM CSS classes from the CSS package — components never contain scoped styles or Shadow DOM stylesheets (unless using `PktShadowElement`).

## Key Packages

| Package | Path | Purpose |
|---|---|---|
| `@oslokommune/punkt-elements` | `packages/elements/` | Lit web components (this package) |
| `@oslokommune/punkt-css` | `packages/css/` | All component CSS (BEM, SCSS) |
| `@oslokommune/punkt-react` | `packages/react/` | React component library |
| `@oslokommune/punkt-assets` | `packages/assets/` | Icons, fonts, animations |
| `shared-types` | `shared-types/` | Shared TypeScript types across packages |
| `shared-utils` | `shared-utils/` | Shared utility functions (date, device, value helpers) |
| Component specs | `component-specs/` | JSON API specifications (source of truth) |

## Core Principles

1. **Light DOM by default** — `PktElement` renders in light DOM for global CSS access and style penetration. Only use `PktShadowElement` when encapsulation is explicitly needed.
2. **External CSS** — all styling via BEM classes from `@oslokommune/punkt-css`. No `static styles` or scoped CSS.
3. **Form participation** — input elements use the `ElementInternals` API for native `<form>` integration.
4. **Slot simulation** — `PktSlotController` provides slot-like content distribution without Shadow DOM.
5. **Framework agnostic** — components work with vanilla JS, React, Vue, and any other framework.

## Base Class Hierarchy

```
LitElement (from 'lit')
  └─ PktShadowElement  (shadow DOM, translations, HMR)
      └─ PktElement  (light DOM override)
          └─ PktInputElement  (form-associated, validation)
              └─ PktOptionsInputElement  (select/combobox options)
```

See [Base Classes](base-classes.md) for full details.
