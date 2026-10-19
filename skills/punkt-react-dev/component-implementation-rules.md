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

1. **`'use client'`** at the top of every file that uses hooks, context, event handlers or browser
   APIs. Leave it out of components that only render markup (the table parts, Heading, Link,
   Stepper): in Next.js App Router they then render as Server Components. Keep it on a component
   that reads a `window.pkt*` global and passes the value on as a prop (`PktButton`); as a Server
   Component the value would be fixed at server render and ignore an override set in the browser.
   Never use `export *` in a `'use client'` file, since Next.js rejects it.
   `src/components/server-components.test.ts` guards this, see [testing](testing.md).
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

## Form field accessibility

`aria-describedby` must sit on the control itself — on a `<label>` or a wrapping element it does
not reach the field. Build the ids with `describedByIds` from `shared-utils/forms/described-by`,
using the same base you pass as the wrapper's `forId`:

```tsx
const describedBy = describedByIds({
  id: inputId,
  hasHelptext: !!helptext,
  hasCounter: showCounter,
  ariaDescribedby,
})
```

`PktInputWrapper` emits `aria-describedby` itself only when `hasFieldset` is set, where it
describes the group with the helptext and, when `hasError` and `errorMessage` are set, the error
message (`{forId}-error`). Otherwise the component alone is responsible for its control, and
forgetting it fails silently — the helptext still renders and looks correct.

With `hasFieldset` and `useWrapper={false}`, the wrapper renders a visually hidden `<legend>`, not
a `<label>`: a label cannot name a fieldset.

Composite controls need it on every focusable part: `PktTimepicker` puts the same value on both
spinbuttons, and `PktCombobox` puts it on the `role="combobox"` element in both the select-only
(a `div`) and text-input (an `input`) modes.

Keep the prop surface identical to Elements. `ariaDescribedby` was missing from `IPktCombobox`
long after Elements had inherited it from `PktInputElement`, so the prop silently did nothing in
React — worth checking against the Elements class when you touch a form component.

## Server rendering

Components must render with `renderToString` in Node, where `window`, `document` and DOM classes
like `HTMLInputElement` do not exist. The build has no SSR shim, so nothing papers over a stray
`window` read. `src/components/ssr.test.tsx` renders every exported component this way.

- Use browser globals in effects and event handlers, not during render. Default parameters,
  `useState` initializers and `useMemo` all run during render.
- When a render-time read is unavoidable, guard it and fall back to the value the browser gets by
  default, so server and client markup match:

  ```tsx
  loadingAnimationPath = typeof window !== 'undefined' ? window.pktAnimationPath : ANIMATION_DEFAULT_PATH
  ```

  Falling back to `undefined` or `0` changes the server markup and causes hydration mismatches.
- A value the server cannot know (viewport or element width) must also use the server value
  during hydration, then switch. `useSyncExternalStore(subscribeNever, () => false, () => true)`
  is `true` on the server and while hydrating, and React renders again with `false` right after.
  See `useElementWidth`.
- Module-level writes to `window.pkt*` globals go behind `typeof window !== 'undefined'`.
- Declare each `window.pkt*` global (`declare global { interface Window { … } }`) in the module
  that uses it, never only in a test file. Test files are left out of the type declarations, so a
  global declared only there breaks the build's type output.
- Module-level code is only for those defaults, and sits in the module that reads the value. The
  ESM build is one file per module with `sideEffects: false`, so bundlers drop every module nobody
  imports, module-level code included.
- Generated ids come from `useGeneratedId(idProp)`. Never create random values (`uuidish()`,
  `Math.random()`, `Date.now()`) during render: the server and the client get different values,
  and React does not patch mismatched attributes when hydrating.
- A DOM API needed inside a callback is looked up when the callback runs. Example:
  `setNativeInputValue` in `src/utils/nativeInputValue.ts`.

## Forbidden Patterns

- **Avoid `forwardRef` for simple ref forwarding** — use `ref` as a regular prop instead (see template above). `forwardRef` is acceptable when `useImperativeHandle` requires a custom handle type.
- **No `displayName`** — unless using `forwardRef`, where it should be set for dev tools clarity.
- **No inline styles** — all styling via BEM classes from `@oslokommune/punkt-css`.
- **No CSS/SCSS files in component directories** — styles live in `packages/css/`.
- **No `import React from 'react'`** or `import * as React` — use named imports only (`import { useState, useEffect } from 'react'`). This is enforced by ESLint.
- **No `console.log`** or `alert()` — enforced by ESLint (`no-console: 'error'`, `no-alert: 'error'`).
