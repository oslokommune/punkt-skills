# Select

Select lets the user choose one option from a dropdown list. It uses the native HTML `<select>` element under the hood, wrapped with Punkt's standard label, help text, and error handling.

## Availability

| Package        | Available | Tag / Import                                                                                   |
| -------------- | --------- | ---------------------------------------------------------------------------------------------- |
| React          | Yes       | `<PktSelect>` — `import { PktSelect } from '@oslokommune/punkt-react'`                         |
| Elements       | Yes       | `<pkt-select>` — `import '@oslokommune/punkt-elements/dist/pkt-select.js'`                     |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/19/elements/pkt-select.js" type="module">` |

Dark mode: Yes

## Usage guidelines

**Use select when:**

- The user needs to choose one option from a predefined list
- There are more than 5–7 options (fewer options are better suited for Radio Button)

**Avoid select when:**

- The user can choose multiple options — use Combobox or Checkbox instead
- The user needs to search or filter — use Combobox instead
- There are very few options (2–4) — use Radio Button instead

## Props / Attributes

| Prop (React)             | Attribute (Elements)     | Type    | Default        | Description                                 |
| ------------------------ | ------------------------ | ------- | -------------- | ------------------------------------------- |
| `label`                  | `label`                  | string  | **(required)** | Label displayed above the field. Required in React; Elements does not enforce it, but the field then has no accessible name |
| `required`               | `required`               | boolean | `false`        | Field must have a value before the form can be submitted |
| `name`                   | `name`                   | string  | —              | Form field name. Falls back to `id`         |
| `form`                   | `form`                   | string  | —              | `id` of the `<form>` the field belongs to, when it sits outside it |
| `id`                     | `id`                     | string  | **(required)** | Unique identifier                           |
| `value`                  | `value`                  | string  | —              | Selected value. React also accepts the native `<select>` types (number, string array) |
| —                        | `options`                | `IPktSelectOption[]` | — | Elements only. Options as objects (`value`, `label`, `selected?`, `disabled?`, `hidden?`) instead of `<option>` children; wins if both are set |
| `helptext`               | `helptext`               | string  | —              | Help text below the label                   |
| `helptextDropdown`       | `helptextDropdown`       | string  | —              | Expandable help text content                |
| `helptextDropdownButton` | `helptextDropdownButton` | string  | —              | Deprecated — use `strings`. See [Strings](strings.md) |
| `hasError`               | `hasError`               | boolean | `false`        | Shows error state                           |
| `errorMessage`           | `errorMessage`           | string  | —              | Error message shown below the field         |
| `disabled`               | `disabled`               | boolean | `false`        | Disables the field                          |
| `inline`                 | `inline`                 | boolean | `false`        | Display inline with page content            |
| `fullwidth`              | `fullwidth`              | boolean | `false`        | Field takes full width                      |
| `useWrapper`             | `useWrapper`             | boolean | `true`         | Wrap the field in the input wrapper         |
| `hasFieldset`            | `hasFieldset`            | boolean | —              | Use fieldset and legend instead of label    |
| `inputSize`              | `input-size`             | `"xsmall"` \| `"small"` \| `"medium"` | `"medium"`     | Field height                                |
| `requiredTag`            | `requiredTag`            | boolean | `false`        | Show "Required" tag next to label           |
| `requiredText`           | `requiredText`           | string  | —              | Deprecated — use `strings`. See [Strings](strings.md)                    |
| `optionalTag`            | `optionalTag`            | boolean | `false`        | Show "Optional" tag next to label           |
| `optionalText`           | `optionalText`           | string  | —              | Deprecated — use `strings`. See [Strings](strings.md)                    |
| `tagText`                | `tagText`                | string  | —              | Custom tag text next to label               |
| `ariaDescribedby`        | `aria-describedby`       | string  | —              | ID of the element that describes this field |
| `ariaLabelledby`         | `aria-labelledby`        | string  | —              | ID of the element that labels this field    |

## Slots

| Slot    | Description                        |
| ------- | ---------------------------------- |
| default | `<option>` elements for the select |

## Accessibility

- Always provide a label connected to the select field
- Use help text and error messages connected via `aria-describedby` (handled automatically by the wrapper)
- Keyboard: open with Enter/Space, navigate with arrow keys, select with Enter

## Examples

### React

```jsx
import { PktSelect } from '@oslokommune/punkt-react'

{
  /* Basic select */
}
;<PktSelect label="District" name="district" id="district">
  <option value="">Choose district...</option>
  <option value="frogner">Frogner</option>
  <option value="gamle-oslo">Gamle Oslo</option>
  <option value="grorud">Grorud</option>
</PktSelect>

{
  /* With error */
}
;<PktSelect
  label="Category"
  name="category"
  id="category"
  requiredTag
  hasError
  errorMessage="Please select a category"
>
  <option value="">Choose...</option>
  <option value="a">Category A</option>
  <option value="b">Category B</option>
</PktSelect>
```

### Elements

```html
<pkt-select label="District" name="district" id="district">
  <option value="">Choose district...</option>
  <option value="frogner">Frogner</option>
  <option value="gamle-oslo">Gamle Oslo</option>
  <option value="grorud">Grorud</option>
</pkt-select>
```
