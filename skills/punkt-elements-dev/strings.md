# UI Strings

All UI text in the design system is defined in `shared-strings/nb.ts` and looked up from
there. Never write a Norwegian string literal in a component — `npm run check-strings`
fails on it in CI.

## The catalogue

`shared-strings/nb.ts` is exactly two levels deep: namespace → key → `string` or `string[]`.

```ts
export const nb = {
  searchinput: {
    placeholder: 'Søk…',
    submit: 'Søk',
  },
} satisfies TPktStringsShape
```

Rules:

- Never functions. Values with variables use `{name}` placeholders.
- Plurals use a `…One` / `…Other` pair.
- Values must be JSON-serializable, so consumers can override from an HTML attribute.

## Looking up text in a component

`PktShadowElement` gives every element a `pktStrings` controller. It merges the built-in
text, the global store and the component's own `strings` property, and re-renders the
component when `setPktStrings()` is called.

```ts
render() {
  const s = this.pktStrings.get('searchinput')
  return html`<input placeholder=${s.placeholder} />`
}
```

For several namespaces at once:

```ts
const s = this.pktStrings.getAll(['forms', 'validation'])
s.forms.optional
s.validation.valueMissing
```

## Placeholders and plurals

```ts
import { formatString, plural } from 'shared-strings'

formatString(s.rangeUnderflowMin, { min: this.min })
plural(count, s.fileLabelOne, s.fileLabelOther)
```

## Composed components

When an element renders another Punkt element, `strings` must be forwarded — otherwise
the inner element never sees the consumer's override:

```ts
html`<pkt-input-wrapper .strings=${this.strings} ...>`
```

## Text in shared-utils

`shared-utils` must not contain UI text. Functions that need to report something return a
key, and the component looks up the text:

```ts
// shared-utils/combobox/selection-manager.ts
return { userInfoMessage: 'maxSelected', ... }

// combobox.ts
this.pktStrings.get('combobox')[this._userInfoMessage]
```

## Deprecated single-string props

Props such as `optionalText` and `searchPlaceholder` are `@deprecated` but still win over
every string layer. The pattern is:

```ts
/** @deprecated Bruk `strings={{ forms: { optional: '…' } }}` eller `setPktStrings()`. */
@property({ type: String }) optionalText?: string

// in render:
${this.optionalText ?? s.optional}
```

Use `??`, not `||` — that lets a consumer pass an empty string to hide the text.

## Adding a new string

1. Add the key to the right namespace in `shared-strings/nb.ts`.
2. Look it up with `this.pktStrings.get(...)` in the element.
3. Do the same in the React component with `usePktStrings(...)`.
4. Run `npm run check-strings` and `npx vitest run ../../shared-strings` from `packages/elements`.
