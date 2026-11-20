# Checklist: Adding a New Component

1. Read the component spec in `component-specs/{name}.spec.ts` (older ones are still `.json`). A new spec is registered in `component-specs/lib/typed-spec-registry.ts`.
2. Create directory: `packages/react/src/components/{name}/`.
3. Create component file: `{Name}.tsx` following the pure React template in [Component Implementation Rules](component-implementation-rules.md).
4. Implement all props from the spec with correct types and defaults.
5. Map props to BEM classes from `packages/css/`.
6. Create test file: `{Name}.test.tsx` covering rendering, classes, ref, a11y.
7. Export component in `packages/react/src/components/index.ts`.
8. Export interface in `packages/react/src/components/interfaces.ts`.
9. Create a dev page: `src/routes/{name}.tsx` and register it in `src/main.tsx` and `src/routes/root.tsx` (see [Dev Pages](dev-pages.md)).
10. If the component needs props to render anything, add them to `requiredProps` in `src/utils/serverRenderCases.tsx`. The SSR and hydration tests render every exported component.
11. Add a conformance test in `packages/conformance/src/tests/{name}.conformance.test.tsx` so React and Elements render the same markup.
12. Run `npm run lint` and `npm run test` from `packages/react/`.
