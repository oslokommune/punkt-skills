# Directory Structure

## Package root (`packages/elements/`)

```
packages/elements/
├── src/
│   ├── index.ts                  # Main entry: re-exports components + types
│   ├── base-elements/            # Base classes
│   │   ├── element.ts            # PktShadowElement + PktElement
│   │   ├── element-with-slot.ts  # PktElementWithSlot (slot content)
│   │   ├── input-element.ts      # PktInputElement (form inputs)
│   │   └── options-input-element.ts  # PktOptionsInputElement (select/combobox)
│   ├── components/               # All components
│   │   ├── index.ts              # Barrel export for all components + types
│   │   ├── button/
│   │   │   ├── button.ts         # Implementation
│   │   │   ├── button.test.ts    # Tests
│   │   │   └── index.ts          # Exports
│   │   ├── select/
│   │   ├── tabs/
│   │   │   ├── tabs.ts           # PktTabs (provider)
│   │   │   ├── tabitem.ts        # PktTabItem (consumer)
│   │   │   ├── tabs-context.ts   # Context definition
│   │   │   ├── tabs.test.ts
│   │   │   └── index.ts
│   │   └── ...                   # One directory per component
│   ├── controllers/              # Reactive controllers
│   │   ├── pkt-options-controller.ts   # <option>/<data> children → options
│   │   ├── strings-controller.ts       # pktStrings (shared-strings lookup)
│   │   ├── focus-modality-controller.ts
│   │   └── pkt-slot-utils.ts
│   ├── directives/
│   │   └── slot-content.ts       # slotContent() directive + SlotManager
│   ├── helpers/
│   │   └── converters.ts         # Attribute converters (CSV, date)
│   ├── types/
│   │   ├── index.ts
│   │   ├── size.ts               # TPktSize
│   │   └── typeUtils.ts          # ElementProps<> helper
│   ├── utils/
│   │   └── classutils.ts         # Host class attribute helpers
│   ├── tests/
│   │   ├── test-framework.ts     # createElementTest() factory
│   │   └── component-registry.ts # Tag-to-class mapping for type-safe tests
│   └── docs/                     # Demo/documentation components (excluded from lint)
├── eslint.config.js
├── tsconfig.json
├── vite.config.ts                # Library build (one entry per component directory)
├── vite.config-app.ts            # Dev app build
├── vitest.config.ts
└── package.json
```

## Component directory convention

Every component follows this structure:

```
component-name/
├── component-name.ts             # Implementation + types + guarded customElement() registration
├── component-name.test.ts        # Tests
├── index.ts                      # Re-exports class + types, default export
└── helpers/                      # Optional: complex components may have helpers
    └── some-helper.ts
```

## Component index.ts pattern

```typescript
import { PktButton } from './button'

export type {
  IPktButton,
  TPktButtonMode,
  TPktButtonSize,
  // ... all exported types
} from './button'

export { PktButton }
export default PktButton
```
