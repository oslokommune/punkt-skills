# Architecture Overview

Punkt is a monorepo design system by Oslo kommune. The React package provides components that consume BEM CSS classes from a separate CSS package — components never contain their own stylesheets or inline styles.

## Key Packages

| Package                       | Path                 | Purpose                                                |
| ----------------------------- | -------------------- | ------------------------------------------------------ |
| `@oslokommune/punkt-react`    | `packages/react/`    | React component library                                |
| `@oslokommune/punkt-css`      | `packages/css/`      | All component CSS (BEM, SCSS)                          |
| `@oslokommune/punkt-elements` | `packages/elements/` | Lit web components                                     |
| `@oslokommune/punkt-assets`   | `packages/assets/`   | Icons, fonts                                           |
| `shared-types`                | `shared-types/`      | Shared TypeScript types across packages                |
| `shared-utils`                | `shared-utils/`      | Shared utility functions (date, device, value helpers) |
| Component specs               | `component-specs/`   | JSON API specifications (source of truth)              |

## Two Component Patterns

1. **Pure React components** (`isPureReact: true` in spec) — native React implementations. Most components.
2. **Lit wrappers** — React wrappers around web components from `punkt-elements`, using `@lit/react`'s `createComponent`.
