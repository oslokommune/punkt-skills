# Component Implementation Rules

## Naming

- **Component name**: `Pkt` prefix + PascalCase. Example: `PktButton`, `PktAccordion`.
- **Interface name**: `IPkt` prefix + ComponentName. Example: `IPktButton`.
- **Shared type name**: `T` prefix. Example: `TAccordionSkin`.
- **File name**: PascalCase matching component. Example: `Button.tsx`, `AccordionItem.tsx`.
- **Directory name**: kebab-case. Example: `button/`, `accordion/`.
- **CSS class prefix**: `pkt-` + kebab-case. Example: `pkt-btn`, `pkt-accordion-item`.

## Component Template (Pure React)

```tsx
'use client'
import { type HTMLAttributes, type ReactNode, type Ref } from 'react'

export interface IPktComponentName extends HTMLAttributes<HTMLDivElement> {
  ref?: Ref<HTMLDivElement>
  size?: 'small' | 'medium' | 'large'
  skin?: 'primary' | 'secondary'
  className?: string
  children?: ReactNode
}

export const PktComponentName = ({
  children,
  className,
  ref,
  size = 'medium',
  skin = 'primary',
  ...props
}: IPktComponentName) => {
  const classes = [
    'pkt-component-name',
    size && `pkt-component-name--${size}`,
    skin && `pkt-component-name--${skin}`,
    className,
  ]
    .filter(Boolean)
    .join(' ')

  return (
    <div {...props} className={classes} ref={ref}>
      {children}
    </div>
  )
}
```

## Required Patterns

1. **`'use client'`** directive at the top of every component file (Next.js compatibility).
2. **`ref` as a regular prop** — prefer accepting `ref?: Ref<HTMLElement>` in the interface and destructuring it like any other prop (the React 19 pattern, supported from React 18.3.1+). However, `forwardRef` is allowed when `useImperativeHandle` is needed to expose an imperative handle (e.g., `PktCalendarHandle`), since React 18 strips `ref` from props before the component receives it when a custom handle type is involved. Add `displayName` when using `forwardRef`.
3. **`className` prop** — always include `className?: string` in the interface and append it to the class array so consumers can add custom classes.
4. **Extend native HTML attributes** in the interface (e.g., `ButtonHTMLAttributes<HTMLButtonElement>`).
4. **Spread `...props`** onto the root element to pass through native HTML attributes.
5. **Default prop values** via destructuring defaults, not `defaultProps`.
6. **Array-based class composition**:
   ```tsx
   const classes = [
     'pkt-block',
     modifier && `pkt-block--${modifier}`,
     className,
   ].filter(Boolean).join(' ')
   ```
   For complex components, `classnames` library is also available.

## Forbidden Patterns

- **Avoid `forwardRef` for simple ref forwarding** — use `ref` as a regular prop instead (see template above). `forwardRef` is acceptable when `useImperativeHandle` requires a custom handle type.
- **No `displayName`** — unless using `forwardRef`, where it should be set for dev tools clarity.
- **No inline styles** — all styling via BEM classes from `@oslokommune/punkt-css`.
- **No CSS/SCSS files in component directories** — styles live in `packages/css/`.
- **No `import React from 'react'`** or `import * as React` — use named imports only (`import { useState, useEffect } from 'react'`). This is enforced by ESLint.
- **No `console.log`** or `alert()` — enforced by ESLint (`no-console: 'error'`, `no-alert: 'error'`).
