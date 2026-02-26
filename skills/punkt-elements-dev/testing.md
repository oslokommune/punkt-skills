# Testing

## Framework

- **Vitest** with jsdom environment
- **@testing-library/jest-dom** for DOM matchers
- **@testing-library/dom** for `fireEvent`
- **jest-axe** for accessibility testing

## Configuration

**File:** `vitest.config.ts`

```typescript
{
  test: {
    environment: 'jsdom',
    setupFiles: ['./setupTests.ts'],
    globals: true,
  }
}
```

**File:** `tsconfig.test.json` — relaxes `noUnusedLocals`, `noUnusedParameters`, and `noImplicitAny` for tests.

## Test file convention

- Co-located with component: `button/button.test.ts`
- Complex components may have multiple test files: `calendar.core.test.ts`, `calendar.accessibility.test.ts`

## Test framework

**File:** `src/tests/test-framework.ts`

### `createElementTest()`

The core factory function for creating test elements:

```typescript
export const createElementTest = async <T extends PktElement, C extends BaseTestConfig>(
  customElementName: CustomElementName,
  config: C = {} as C,
): Promise<{
  container: HTMLElement
  element: T
}>
```

It:
1. Creates a container `<div>`
2. Builds HTML with properly escaped attributes from config
3. Appends to `document.body`
4. Waits for `customElements.whenDefined()`
5. Waits for `element.updateComplete`
6. Returns `{ container, element }`

### Component-specific test helpers

Each test file defines a typed wrapper around `createElementTest`:

```typescript
import { createElementTest, BaseTestConfig } from '../../tests/test-framework'
import { CustomElementFor } from '../../tests/component-registry'
import './button'  // Ensure the component is registered

export interface ButtonTestConfig extends BaseTestConfig {
  size?: string
  skin?: string
  variant?: string
  disabled?: boolean
  isLoading?: boolean
  content?: string
}

export const createButtonTest = async (config: ButtonTestConfig = {}) => {
  const { container, element } = await createElementTest<
    CustomElementFor<'pkt-button'>,
    ButtonTestConfig
  >('pkt-button', config)

  return {
    container,
    button: element,
  }
}
```

### Component registry

**File:** `src/tests/component-registry.ts`

Maps custom element tag names to their TypeScript classes for type inference:

```typescript
export const customElementComponentRegistry = {
  'pkt-button': PktButton,
  'pkt-select': PktSelect,
  // ...
}

export type CustomElementName = keyof typeof customElementComponentRegistry
export type CustomElementFor<T extends CustomElementName> = InstanceType<
  (typeof customElementComponentRegistry)[T]
>
```

When adding a new component, register it here for type-safe testing.

## Test structure

```typescript
import '@testing-library/jest-dom'
import { axe, toHaveNoViolations } from 'jest-axe'
import { fireEvent } from '@testing-library/dom'
import { vi } from 'vitest'

expect.extend(toHaveNoViolations)

afterEach(() => {
  document.body.innerHTML = ''
})

describe('PktButton', () => {
  describe('Rendering', () => {
    test('renders without errors', async () => {
      const { button } = await createButtonTest({ content: 'Click me' })
      expect(button).toBeInTheDocument()
    })
  })

  describe('Properties', () => { ... })
  describe('States', () => { ... })
  describe('Events', () => { ... })
  describe('Accessibility', () => { ... })
})
```

## Testing patterns

### Property testing
```typescript
test('applies size correctly', async () => {
  const { button } = await createButtonTest({ size: 'small', content: 'Test' })
  await button.updateComplete

  expect(button.size).toBe('small')
  const nativeButton = button.querySelector('button')
  expect(nativeButton).toHaveClass('pkt-btn--small')
})
```

### Dynamic property updates
```typescript
test('updates on property change', async () => {
  const { button } = await createButtonTest({ content: 'Test' })

  button.isLoading = true
  await button.updateComplete

  const nativeButton = button.querySelector('button')
  expect(nativeButton).toHaveClass('pkt-btn--isLoading')
})
```

### Event testing
```typescript
test('dispatches click event', async () => {
  const { button } = await createButtonTest({ content: 'Test' })
  const clickSpy = vi.fn()
  button.addEventListener('click', clickSpy)

  const nativeButton = button.querySelector('button')
  fireEvent.click(nativeButton!)

  expect(clickSpy).toHaveBeenCalledTimes(1)
})
```

### Custom event testing
```typescript
test('dispatches tab-selected on click', async () => {
  const eventListener = vi.fn()
  tabs.addEventListener('tab-selected', eventListener)

  button?.click()

  expect(eventListener).toHaveBeenCalled()
  expect(eventListener.mock.calls[0][0].detail.index).toBe(1)
})
```

### Slot content testing
```typescript
test('renders slot content', async () => {
  const { element } = await createElementTest('pkt-alert', {
    content: '<p>Alert message with <a href="#">a link</a>.</p>',
  })

  const content = element.querySelector('p')
  expect(content).toBeInTheDocument()
})
```

### Light DOM querying
```typescript
// Light DOM — query directly on the element
const nativeButton = button.querySelector('button')
const icon = button.querySelector('pkt-icon')
```

### Shadow DOM querying
```typescript
// Shadow DOM — use shadowRoot
const div = accordion.shadowRoot?.querySelector('.pkt-accordion')
```

### Accessibility testing (jest-axe)

**Every component must have at least one axe test:**

```typescript
test('has no WCAG violations', async () => {
  const { container } = await createButtonTest({ content: 'Click me' })
  const results = await axe(container)
  expect(results).toHaveNoViolations()
})
```

Test multiple variants:
```typescript
test('accessible when disabled', async () => {
  const { container } = await createButtonTest({ disabled: true, content: 'Disabled' })
  expect(await axe(container)).toHaveNoViolations()
})
```

### Waiting for updates
```typescript
// Wait for Lit render cycle
await element.updateComplete

// Wait for async side effects (e.g., keyboard navigation)
await new Promise((resolve) => setTimeout(resolve, 50))

// Wait for custom element to be defined
await customElements.whenDefined('pkt-tabs')
```

### Console mocking
```typescript
import { setupConsoleMocking, restoreConsoleMocking } from '../../tests/test-framework'

beforeEach(() => setupConsoleMocking({ mockError: true }))
afterEach(() => restoreConsoleMocking())
```

## Commands

```bash
npm test          # Watch mode
npm run test:run  # Single run
npm run test:ui   # Vitest UI
```
