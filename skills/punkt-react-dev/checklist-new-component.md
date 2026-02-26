# Checklist: Adding a New Component

1. Read the component spec in `component-specs/{name}.json`.
2. Create directory: `packages/react/src/components/{name}/`.
3. Create component file: `{Name}.tsx` following the pure React template in [Component Implementation Rules](component-implementation-rules.md).
4. Implement all props from the spec with correct types and defaults.
5. Map props to BEM classes from `packages/css/`.
6. Create test file: `{Name}.test.tsx` covering rendering, classes, ref, a11y.
7. Export component in `packages/react/src/components/index.ts`.
8. Export interface in `packages/react/src/components/interfaces.ts`.
9. Create a dev page: `src/routes/{name}.tsx` and register it in `src/main.tsx` and `src/routes/root.tsx` (see [Dev Pages](dev-pages.md)).
10. Run `npm run lint` and `npm run test` from `packages/react/`.
