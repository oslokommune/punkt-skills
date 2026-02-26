# TypeScript & Types

## TypeScript configuration

Key `tsconfig.json` settings for Lit:
- `experimentalDecorators: true` — required for `@property`, `@customElement`, etc.
- `useDefineForClassFields: false` — required for Lit decorators to work correctly
- `target: "ES2020"`
- `strict: true`
- `noUnusedLocals: true`, `noUnusedParameters: true`

## Path aliases

```json
{
  "@/*": ["src/*"],
  "shared-types": ["../../shared-types"],
  "shared-types/*": ["../../shared-types/*"],
  "shared-utils/*": ["../../shared-utils/*"],
  "pkt/*": ["node_modules/@oslokommune/punkt-css/dist/*"],
  "pktAssets/*": ["node_modules/@oslokommune/punkt-assets/dist/*"],
  "componentSpecs/*": ["../../component-specs/*"],
  "punkt-elements": ["src/components/index.ts"]
}
```

Use `@/` prefix for internal imports:
```typescript
import { PktElement } from '@/base-elements/element'
import { PktSlotController } from '@/controllers/pkt-slot-controller'
import '@/components/icon'
```

## Component type patterns

### Interface for public API

Every component exports an interface describing its public props:

```typescript
export interface IPktButton {
  iconName?: PktIconName
  size?: TPktButtonSize
  skin?: TPktButtonSkin
  variant?: TPktButtonVariant
  disabled?: Booleanish
  isLoading?: Booleanish
}
```

The class both extends the base with `<IPktButton>` and `implements IPktButton`:

```typescript
@customElement('pkt-button')
export class PktButton extends PktElement<IPktButton> implements IPktButton {
```

### Type aliases for string unions

```typescript
export type TPktButtonSkin = 'primary' | 'secondary' | 'tertiary'
export type TPktButtonSize = 'small' | 'medium' | 'large'
export type TPktButtonVariant = 'label-only' | 'icon-left' | 'icon-right' | 'icon-only'
```

Convention: `T` prefix + `Pkt` + ComponentName + PropertyName.

### ElementProps utility

**File:** `src/types/typeUtils.ts`

```typescript
export type ElementProps<Element, PropKeys extends keyof Element> = Partial<Pick<Element, PropKeys>>
```

Used internally by base classes to define the Vue `$props` type:

```typescript
type Props = ElementProps<PktInputElement, 'disabled' | 'required' | 'name' | ...>
```

## Shared types

From `shared-types`:
- `Booleanish` — `true | false | 'true' | 'false'`
- `booleanishConverter` — Lit attribute converter for Booleanish values
- `THTMLButtonType` — `'button' | 'submit' | 'reset'`
- `TAriaLive` — ARIA live region types

From `shared-utils`:
- `uuidish()` — generates unique IDs for form elements
- `parseISODateString()`, `formatISODate()`, `newDate()`, `newDateYMD()` — date utilities
- `valueToArray()` — value normalization

## HTMLElementTagNameMap declaration

For components that extend native element interfaces, add a global type declaration:

```typescript
declare global {
  interface HTMLElementTagNameMap {
    'pkt-select': PktSelect & HTMLSelectElement
  }
}
```

This provides type inference when using `document.querySelector('pkt-select')`.

## Export patterns

### Component barrel (index.ts)

```typescript
import { PktButton } from './button'

export type {
  IPktButton,
  TPktButtonMode,
  TPktButtonSize,
  TPktButtonSkin,
} from './button'

export { PktButton }
export default PktButton
```

### Package barrel (components/index.ts)

Exports all components and their types:

```typescript
export { PktButton } from '@/components/button'
export type { IPktButton, TPktButtonSkin, ... } from '@/components/button'
```

### Top-level entry (src/index.ts)

```typescript
export * from './components'
export * from './types'
```
