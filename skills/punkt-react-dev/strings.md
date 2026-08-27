# UI Strings

All UI text in the design system is defined in `shared-strings/nb.ts` and looked up from
there. Never write a Norwegian string literal in a component — `npm run check-strings`
fails on it in CI.

## The catalogue

`shared-strings/nb.ts` is exactly two levels deep: namespace → key → `string` or `string[]`.
It is shared with Elements, so a string added there applies to both frameworks.

Rules:

- Never functions. Values with variables use `{name}` placeholders.
- Plurals use a `…One` / `…Other` pair.
- Values must be JSON-serializable.

## Looking up text in a component

```tsx
import { usePktStrings } from '../../hooks/usePktStrings'

export const PktSearchInput = forwardRef(({ placeholder, strings, ...rest }, ref) => {
  const s = usePktStrings(['searchinput'], strings)
  return <input placeholder={placeholder ?? s.searchinput.placeholder} />
})
```

The hook takes the namespaces the component uses plus the component's own `strings` prop.
It subscribes to the global store, so `setPktStrings()` after mount triggers a re-render,
and it reads the nearest `PktStringsProvider`.

## The `strings` prop

Every component that renders text should accept a `strings` prop:

```ts
import type { TPktStringsOverride } from 'shared-strings'

export interface IPktSearchInput {
  /** Overstyrer tekster for denne komponenten. */
  strings?: TPktStringsOverride
}
```

Composed components forward it to their subcomponents, the same way `optionalText` is
already forwarded to `PktInputWrapper`.

## Placeholders and plurals

```ts
import { formatString, plural } from 'shared-strings'

formatString(s.validation.rangeUnderflowMin, { min })
plural(count, s.fileupload.fileLabelOne, s.fileupload.fileLabelOther)
```

## Text outside a component

Plain functions cannot call hooks. Pass the resolved strings in as a parameter:

```ts
function validateTimeValue(value: string, opts: IOpts, messages: TPktStrings['validation']) {
  if (opts.required && !value) return { valid: false, message: messages.valueMissing }
}

// in the hook:
const catalog = usePktStrings(['timepicker', 'validation'], stringsProp)
validateTimeValue(value, opts, catalog.validation)
```

## Deprecated single-string props

Props such as `optionalText` and `placeholder` are `@deprecated` but still win over every
string layer. Remove the literal default in the destructuring and use `??`:

```tsx
// before
({ optionalText = 'Valgfritt' }) => <span>{optionalText}</span>

// after
({ optionalText, strings }) => {
  const s = usePktStrings(['forms'], strings)
  return <span>{optionalText ?? s.forms.optional}</span>
}
```

Use `??`, not `||` — that lets a consumer pass an empty string to hide the text.

## Adding a new string

1. Add the key to the right namespace in `shared-strings/nb.ts`.
2. Look it up with `usePktStrings(...)` in the React component.
3. Do the same in the Elements component with `this.pktStrings.get(...)`.
4. Run `npm run check-strings` and `npx vitest run ../../shared-strings` from `packages/elements`.
