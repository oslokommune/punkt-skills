# Strings and overrides

All UI text in Punkt comes from a single shared catalogue that both Elements and React
read from. The built-in language is Norwegian Bokmål. You can override text globally for
the whole application, for a subtree (React only), or on a single component.

## Global override

Call `setPktStrings()` once at startup. It applies to both Elements and React on the same
page, and already-mounted components re-render.

```ts
import { setPktStrings } from '@oslokommune/punkt-react'
// or: import { setPktStrings } from '@oslokommune/punkt-elements'

setPktStrings({
  forms: { optional: 'Frivillig' },
  validation: { valueMissing: 'Du må fylle ut dette feltet' },
})
```

Repeated calls merge. Keys you don't mention keep their default value.

## Per component

Every component that renders text takes a `strings` prop that wins over the global
override. In Elements every element accepts `strings`, including those with no text of
their own, because it comes from the base class.

```tsx
<PktSearchInput
  id="sok"
  strings={{ searchinput: { placeholder: 'Finn ansatt…' } }}
/>
```

```html
<pkt-searchinput
  id="sok"
  strings='{"searchinput":{"placeholder":"Finn ansatt…"}}'
></pkt-searchinput>
```

In Elements, `strings` can be set as a JavaScript property or as JSON in the attribute.
When a component is composed of several elements (for example `pkt-textinput`, which
renders `pkt-input-wrapper`), `strings` is forwarded inwards automatically.

## Per subtree (React only)

```tsx
import { PktStringsProvider } from '@oslokommune/punkt-react'

<PktStringsProvider strings={{ forms: { optional: 'Kan stå tomt' } }}>
  <PktTextinput id="navn" name="navn" label="Navn" optionalTag />
</PktStringsProvider>
```

Providers can be nested — an inner provider merges on top of the outer one.

## Resolution order

Lowest to highest precedence:

1. Built-in Bokmål text
2. `setPktStrings()`
3. `PktStringsProvider` (React)
4. The component's `strings` prop
5. Deprecated single-string props such as `optionalText` and `searchPlaceholder`

## Placeholders

Text with variables uses `{name}` placeholders. Keep the placeholders when you override,
otherwise the value is missing from the rendered string.

```ts
setPktStrings({
  validation: { rangeUnderflowMin: 'Kan ikke være lavere enn {min}' },
})
```

Plurals use a `…One` / `…Other` pair:

```ts
setPktStrings({
  fileupload: { fileLabelOne: 'dokument', fileLabelOther: 'dokumenter' },
})
```

## Namespaces

The catalogue is two levels: namespace → key.

| Namespace     | Contents                                                    |
| ------------- | ----------------------------------------------------------- |
| `generic`     | `from`, `to`, `close`                                        |
| `forms`       | `optional`, `required`, `readMore`                           |
| `validation`  | Validation messages for all form fields                      |
| `dates`       | Month and day names, `week`, `month`, `year`                 |
| `calendar`    | `prevMonth`, `nextMonth`                                     |
| `datepicker`  | `openPicker`, `deleteDate`                                   |
| `timepicker`  | `hours`, `minutes`, `openPicker`, `prevTime`, `nextTime`     |
| `searchinput` | `placeholder`, `submit`                                      |
| `listbox`     | `searchLabel`, `searchPlaceholder`                           |
| `combobox`    | Status messages for selection, search and the max limit      |
| `fileupload`  | Drop zone, validation and screen reader announcements        |
| `backlink`    | `label`                                                      |
| `breadcrumbs` | `ariaLabel`                                                  |
| `tag`         | `remove`                                                     |
| `header`      | `search`, `menu`, `openMenu`, `openSearch`                   |

All keys and their values live in `shared-strings/nb.ts` in the monorepo, or you can
import the catalogue:

```ts
import { pktStringsNb } from '@oslokommune/punkt-react'
console.log(pktStringsNb.validation)
```

## Deprecated props

Single-string props such as `optionalText`, `requiredText`, `searchPlaceholder`,
`helptextDropdownButton` and the calendar's `dayStrings` / `monthStrings` still work and
win over every string layer. They are marked `@deprecated` and will be removed in a later
major version. Use `strings` or `setPktStrings()` in new code.
