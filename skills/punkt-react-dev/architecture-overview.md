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

## Everything is pure React

Every component is a native React implementation. The package has no runtime dependency on `@oslokommune/punkt-elements`, `lit` or `@lit/react` — check `packages/react/package.json` and you will not find them.

Punkt React is pure React — there are no `@lit/react` wrappers. Do not add any: components are written as React components, including `PktIcon`. Code or tutorials showing `createComponent` do not apply here.

React and Elements are now two independent implementations that share `shared-types`, `shared-utils` and the component specs.
