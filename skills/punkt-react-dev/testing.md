# Testing

## Stack

- **Vitest** (test runner, `vitest run`)
- **@testing-library/react** (rendering, queries)
- **@testing-library/user-event** (interaction simulation)
- **jest-axe** (WCAG accessibility assertions)
- **jsdom** environment

## Test file location

Co-located: `ComponentName.test.tsx` next to `ComponentName.tsx`.

## Test structure

```tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { axe, toHaveNoViolations } from 'jest-axe'
import { createRef } from 'react'
import { PktComponentName } from './ComponentName'

expect.extend(toHaveNoViolations)

describe('PktComponentName', () => {
  // Basic rendering
  test('renders with default props', () => {
    render(<PktComponentName>Content</PktComponentName>)
    expect(screen.getByText('Content')).toBeInTheDocument()
  })

  // CSS classes
  test('applies correct classes for props', () => {
    const { container } = render(
      <PktComponentName size="small" skin="secondary" />
    )
    const el = container.firstChild
    expect(el).toHaveClass('pkt-component-name')
    expect(el).toHaveClass('pkt-component-name--small')
    expect(el).toHaveClass('pkt-component-name--secondary')
  })

  // Custom className passthrough
  test('supports custom className', () => {
    const { container } = render(
      <PktComponentName className="custom-class" />
    )
    expect(container.firstChild).toHaveClass('custom-class')
  })

  // Ref forwarding
  test('forwards ref correctly', () => {
    const ref = createRef<HTMLDivElement>()
    render(<PktComponentName ref={ref}>Content</PktComponentName>)
    expect(ref.current).toBeInstanceOf(HTMLDivElement)
  })

  // User interaction
  test('handles click events', async () => {
    const onClick = vi.fn()
    render(<PktComponentName onClick={onClick}>Click</PktComponentName>)
    await userEvent.click(screen.getByText('Click'))
    expect(onClick).toHaveBeenCalledTimes(1)
  })

  // Accessibility
  describe('accessibility', () => {
    it('has no axe violations', async () => {
      const { container } = render(
        <PktComponentName>Content</PktComponentName>
      )
      const results = await axe(container)
      expect(results).toHaveNoViolations()
    })
  })
})
```

## What to Test

1. Default rendering and prop application
2. BEM class generation for all prop variants
3. Custom `className` passthrough
4. Ref forwarding with `createRef`
5. User interactions (click, keyboard, focus)
6. Controlled vs uncontrolled behavior (if applicable)
7. Accessibility (jest-axe, ARIA attributes)
8. Context consumption (for compound components)

## Test utilities

Custom render available at `src/utils/test-utils.tsx` — re-exports `@testing-library/react` with `asFragmentWithoutComments()`.

## Running tests

```bash
# From packages/react/
npm run test          # Single run
npm run test:watch    # Watch mode
```
