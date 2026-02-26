# Checklist: Adding a New Element

1. Read the component spec in `component-specs/{name}.json` for required props, types, and modifiers.
2. Decide the base class:
   - `PktElement` for display components
   - `PktInputElement` for form inputs
   - `PktOptionsInputElement` for select/combobox-style inputs
   - `PktShadowElement` only if Shadow DOM encapsulation is explicitly needed
3. Create the component directory: `src/components/{name}/`.
4. Create the implementation file `{name}.ts`:
   - Define types and interface (`IPkt*`, `T*`)
   - Use `@customElement('pkt-{name}')` decorator
   - Extend the correct base class with `<Interface>` and `implements Interface`
   - Initialize `PktSlotController` in constructor if the component accepts children
   - Initialize `PktOptionsSlotController` if the component accepts `<option>` children (set `slotController.skipOptions = true`)
   - Use `classMap()` for dynamic CSS classes
   - Use `ref()` for DOM references
   - Add `export default` at the bottom
5. Create `index.ts`:
   - Import the class
   - Re-export the class + all types
   - Add default export
6. Register the component in the barrel export: `src/components/index.ts`.
7. Register the component in the test registry: `src/tests/component-registry.ts`.
8. Create `{name}.test.ts`:
   - Define a typed test config extending `BaseTestConfig`
   - Create a component-specific test helper using `createElementTest()`
   - Test rendering, properties, states, events
   - Include at least one `axe` accessibility test
9. Add the build entry in `vite.config.ts` under `lib.entry`.
10. Verify the CSS classes exist in `@oslokommune/punkt-css` — if not, create them using the [css-dev skill](../punkt-css-dev.md).
11. Run `npm run build` to verify compilation.
12. Run `npm run test:run` to verify tests pass.
