# TypeScript

## Config

- **Strict mode** enabled.
- **Path aliases** in `tsconfig.json`:
  - `@/*` → `src/*`
  - `shared-types` → `../../shared-types`
  - `pkt/*` → `node_modules/@oslokommune/punkt-css/dist/*`

## Type Organization

1. **Component-level**: Interface defined and exported in the component file itself.
2. **Shared types**: Cross-package types in `shared-types/` at monorepo root.
3. **Package interfaces**: `src/interfaces/` for Lit wrapper types (e.g., `IPktElements.tsx`).

## Common Type Patterns

```tsx
// Extend native HTML attributes
interface IPktButton extends ButtonHTMLAttributes<HTMLButtonElement> { }

// Union types for variants
size?: 'small' | 'medium' | 'large'

// Optional ReactNode children
children?: ReactNode | ReactNode[]
helptext?: string | ReactNode | ReactNode[]

// Controlled + uncontrolled
defaultOpen?: boolean    // Uncontrolled
isOpen?: boolean         // Controlled
```
