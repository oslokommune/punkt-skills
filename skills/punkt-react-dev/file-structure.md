# File Structure

## Component directory layout
```
packages/react/src/components/{component-name}/
  ComponentName.tsx          # Main component
  ComponentName.test.tsx     # Co-located tests
  types.ts                   # (optional) Complex type definitions
  SubComponent.tsx           # (optional) Compound component parts
```

## Barrel exports

All components are exported through:
- `packages/react/src/components/index.ts` — component exports
- `packages/react/src/components/interfaces.ts` — type-only exports
- `packages/react/src/index.ts` — re-exports `./components`

When adding a new component, register it in both `index.ts` and `interfaces.ts`.
