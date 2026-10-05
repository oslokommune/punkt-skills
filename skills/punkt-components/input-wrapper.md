# Input Wrapper

Input Wrapper provides the standard label, help text, error messages, character counter, and optional/required tags around a form element. Most Punkt form components (Text Input, Textarea, Select, etc.) use Input Wrapper internally via the `useWrapper` prop. Use it directly when wrapping custom or third-party form elements.

## Availability

| Package        | Available | Tag / Import                                                                                      |
| -------------- | --------- | ------------------------------------------------------------------------------------------------- |
| React          | Yes       | `<PktInputWrapper>` — `import { PktInputWrapper } from '@oslokommune/punkt-react'`                |
| Elements       | Yes       | `<pkt-input-wrapper>` — `import '@oslokommune/punkt-elements/dist/pkt-input-wrapper.js'`          |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/19/elements/pkt-input-wrapper.js" type="module">` |

Dark mode: No

## Usage guidelines

**Use Input Wrapper when:**

- You're wrapping a custom or third-party form element that needs standard Punkt labels, help text, and error messages
- You need a `fieldset`/`legend` wrapper for a group of checkboxes or radio buttons

**You usually don't need to use it directly** because Punkt form components (Text Input, Textarea, Select, Datepicker, etc.) include it via the `useWrapper` prop (enabled by default).

## Props / Attributes

| Prop (React)             | Attribute (Elements)     | Type                                  | Default          | Description                                                                                                                      |
| ------------------------ | ------------------------ | ------------------------------------- | ---------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `forId`                  | `forId`                  | string                                | **(required)**   | ID of the form element being wrapped                                                                                             |
| `label`                  | `label`                  | string                                | **(required)**   | Label text for the form element                                                                                                  |
| `helptext`               | `helptext`               | string                                | —                | Help text below the label                                                                                                        |
| `helptextDropdown`       | `helptextDropdown`       | string                                | —                | Expandable help text content                                                                                                     |
| `helptextDropdownButton` | `helptextDropdownButton` | string                                | `"Les mer"`      | Button text for expandable help                                                                                                  |
| `ariaDescribedby`        | `ariaDescribedby`        | string                                | —                | ID of the element that describes the form element                                                                                |
| `counter`                | `counter`                | boolean                               | `false`          | Show character counter                                                                                                           |
| `counterCurrent`         | `counterCurrent`         | number                                | —                | Current character count                                                                                                          |
| `counterMaxLength`       | `counterMaxLength`       | number                                | —                | Maximum character count                                                                                                          |
| `counterPosition`        | `counterPosition`        | `"top"` \| `"bottom"`                 | `"bottom"`       | Position of the counter                                                                                                          |
| `optionalTag`            | `optionalTag`            | boolean                               | `false`          | Show "Optional" tag next to label                                                                                                |
| `optionalText`           | `optionalText`           | string                                | `"Valgfritt"`    | Deprecated — use `strings`. See [Strings](strings.md)                                                                            |
| `requiredTag`            | `requiredTag`            | boolean                               | `false`          | Show "Required" tag next to label                                                                                                |
| `requiredText`           | `requiredText`           | string                                | `"Må fylles ut"` | Deprecated — use `strings`. See [Strings](strings.md)                                                                            |
| `tagText`                | `tagText`                | string                                | —                | Custom tag text next to label                                                                                                    |
| `hasError`               | `hasError`               | boolean                               | `false`          | Shows error state                                                                                                                |
| `errorMessage`           | `errorMessage`           | string                                | —                | Error message shown below the field                                                                                              |
| `disabled`               | `disabled`               | boolean                               | `false`          | Disabled state. With `hasFieldset` the fieldset gets `disabled`, which disables every field in it, like `<fieldset disabled>`    |
| `inline`                 | `inline`                 | boolean                               | `false`          | Display inline with page content                                                                                                 |
| `hasFieldset`            | `hasFieldset`            | boolean                               | `false`          | Render as a `fieldset` with `legend` instead of `div`/`label`                                                                    |
| `layout`                 | `layout`                 | `"vertical"` \| `"horizontal"`        | `"vertical"`     | Direction of the options in a group. `horizontal` wraps them in a row. The gap between options follows `size`: 16, 12 or 8px     |
| `useWrapper`             | `useWrapper`             | boolean                               | `true`           | Enable/disable the wrapper                                                                                                       |
| `size`                   | `input-wrapper-size`     | `"small"` \| `"medium"` \| `"xsmall"` | `"medium"`       | Size of label, help text and error message. Checkboxes and radio buttons inside inherit it unless they set their own `inputSize` |
| `role`                   | `role`                   | string                                | `"group"`        | ARIA role for the wrapper element                                                                                                |

## Events

| Event (React)      | Event (Elements) | Description                                                         |
| ------------------ | ---------------- | ------------------------------------------------------------------- |
| `onToggleHelpText` | `toggleHelpText` | Fires when expandable help text opens/closes. `{ isOpen: boolean }` |

## Slots

| Slot       | Description                                                                  |
| ---------- | ---------------------------------------------------------------------------- |
| default    | The form element or group of elements to wrap                                |
| `helptext` | Elements only. Rich help text, as an alternative to the `helptext` attribute |

Slotted helptext can be added or removed after the first render; the wrapper and the field's
`aria-describedby` follow.

## Accessibility

- The label is automatically connected to the form element via `forId`
- Help text and error messages are connected via `aria-describedby` (handled automatically)
- For groups of checkboxes or radio buttons, prefer [Checkbox Group](checkbox-group.md) and [Radio Group](radio-group.md), which render this wrapper with a fieldset. The groups can also hold other fields, such as a text field for an "Other" option. Use `hasFieldset` directly when you need a fieldset without a group component. It renders a `fieldset` with `legend` for proper screen reader grouping
- For short options side by side, use `layout="horizontal"` instead of your own flex container. Tiles keep their fixed width and wrap to a new line
- With `hasFieldset`, the fieldset is described by the help text and, when `hasError` is set, by the error message
- With `hasFieldset` and `useWrapper={false}`, the legend is visually hidden but still names the group
- Input Wrapper itself takes `counter` and `hasFieldset` as plain booleans. The **components** that
  use it (Text Input, Combobox, Datepicker …) treat them as tri-state: unset means "derive a
  sensible default", explicit `false` always turns it off
- The counter's visible number is `aria-hidden`. A separate live region announces only when the
  limit is exceeded, so screen readers aren't told the count on every keystroke
- Error messages should be specific and explain how to fix the issue

## Examples

### React

```jsx
import { PktInputWrapper } from '@oslokommune/punkt-react'

{
  /* Wrapping a custom input */
}
;<PktInputWrapper forId="custom-field" label="Custom field" helptext="Enter a value" requiredTag>
  <input type="text" id="custom-field" name="custom-field" />
</PktInputWrapper>

{
  /* Fieldset for a checkbox group */
}
;<PktInputWrapper
  forId="colors"
  label="Choose colors"
  hasFieldset
  hasError={hasError}
  errorMessage="Please select at least one color"
>
  <PktCheckbox label="Red" name="color" id="color-red" value="red" />
  <PktCheckbox label="Blue" name="color" id="color-blue" value="blue" />
  <PktCheckbox label="Green" name="color" id="color-green" value="green" />
</PktInputWrapper>
```

### Elements

```html
<pkt-input-wrapper forId="custom-field" label="Custom field" helptext="Enter a value">
  <input type="text" id="custom-field" name="custom-field" />
</pkt-input-wrapper>
```
