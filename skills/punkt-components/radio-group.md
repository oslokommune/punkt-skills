# Radio Group

Radio Group wraps a set of radio buttons in a fieldset with label, help text and error message, and gives them a shared name, size and value. Use it for any group of radio buttons.

**When you use the group, the group is the form field.** Put `name`, `value`, `onChange`, `onBlur`, `required` and form-library bindings on the group, not on the radios. The group element has `name`, a readable and writable `value` and `focus()` (React: on `ref`; Elements: on the element), and its events carry the group as `event.target`.

## Availability

| Package        | Available | Tag / Import                                                                                     |
| -------------- | --------- | ------------------------------------------------------------------------------------------------ |
| React          | Yes       | `<PktRadioGroup>` — `import { PktRadioGroup } from '@oslokommune/punkt-react'`                   |
| Elements       | Yes       | `<pkt-radio-group>` — `import '@oslokommune/punkt-elements/dist/pkt-radio-group.js'`             |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/19/elements/pkt-radio-group.js" type="module">` |

Dark mode: Yes

The Elements bundle registers `pkt-radiobutton` too.

## Props / Attributes

| Prop (React)                        | Attribute (Elements)                | Type                                  | Default        | Description                                                                                                         |
| ----------------------------------- | ----------------------------------- | ------------------------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------- |
| `label`                             | `label`                             | string                                | **(required)** | Group label, rendered as the `legend`                                                                               |
| `id`                                | `id`                                | string                                | —              | Group id. Also the fallback `name` for the radios                                                                   |
| `name`                              | `name`                              | string                                | —              | Given to radios without their own `name`. Falls back to `id`, then to a generated name                              |
| `value`                             | `value`                             | string                                | —              | Value of the checked radio, like `RadioNodeList.value`. `''` when none is checked, `'on'` for a radio without value |
| `defaultValue`                      | —                                   | string                                | —              | React only: initially checked value of an uncontrolled group. Elements uses the `value` attribute                   |
| `layout`                            | `layout`                            | `"vertical"` \| `"horizontal"`        | `"vertical"`   | Direction of the radios. The gap follows the size: 16, 12 or 8px                                                    |
| `inputSize`                         | `input-size`                        | `"small"` \| `"medium"` \| `"xsmall"` | `"medium"`     | Size of the group. Radios inherit it unless they set their own                                                      |
| `hasTile`                           | `hasTile`                           | boolean                               | `false`        | Shows every radio as a tile                                                                                         |
| `required`                          | `required`                          | boolean                               | `false`        | Makes the whole group required, met when one radio is checked                                                       |
| `disabled`                          | `disabled`                          | boolean                               | `false`        | Disables every radio, like `<fieldset disabled>`                                                                    |
| `hasError`                          | `hasError`                          | boolean                               | `false`        | Error state on the group and every radio                                                                            |
| `errorMessage`                      | `errorMessage`                      | string                                | —              | Shown below the radios when `hasError` is set                                                                       |
| `helptext`                          | `helptext`                          | string                                | —              | Help text above the radios. Elements also takes a `helptext` slot                                                   |
| `helptextDropdown`                  | `helptextDropdown`                  | string                                | —              | Expandable help text                                                                                                |
| `useWrapper`                        | `useWrapper`                        | boolean                               | `true`         | `false` hides label and help text visually; the group keeps its accessible name                                     |
| `requiredTag` / `optionalTag`       | `requiredTag` / `optionalTag`       | boolean                               | `false`        | Tags next to the label                                                                                              |
| `tagText`                           | `tagText`                           | string                                | —              | Custom tag next to the label                                                                                        |

## Events

| Event (React)   | Event (Elements) | Description                                                                                                  |
| --------------- | ---------------- | ------------------------------------------------------------------------------------------------------------ |
| `onChange`      | `change`         | Fired by the group when the user changes the selection. `event.target` is the group, `event.target.value` its value (`string`) |
| —               | `input`          | Fired by the group just before `change`, with the value already updated                                       |
| `onValueChange` | `value-change`   | The group's new value (`string`). React: a callback. Elements: `CustomEvent` with `detail`                     |
| `onFocus`       | `focus`          | Focus enters the group. Not fired when focus moves between radios                                           |
| `onBlur`        | `blur`           | Focus leaves the group. Not fired when focus moves between radios                                           |

In Elements the radios' own events stop at the group, so a listener on the group or the form gets one event per change, with the group as target. Listeners on a radio itself still fire. In React the radios' own `onChange` still runs. Setting `value` from code fires no events.

## Form libraries

React Hook Form works with every pattern: `register` on the group, `Controller` with `{...field}` spread on the group, `Controller` with `onValueChange={field.onChange}`, or `register` on each radio. Formik-style handlers that read `event.target.name` and `event.target.value` get the group's name and value.

```jsx
<PktRadioGroup label="…" {...register('transport', { required: true })}>
  <PktRadioButton id="a" value="a" label="A" />
  <PktRadioButton id="b" value="b" label="B" />
</PktRadioGroup>

<Controller
  name="transport"
  control={control}
  render={({ field }) => (
    <PktRadioGroup label="…" {...field}>
      <PktRadioButton id="a" value="a" label="A" />
    </PktRadioGroup>
  )}
/>
```

## Rules for the radios

- The group only gives defaults. `name`, `inputSize` and other props set on a radio win
- `hasTile` and `hasError` on the group turn the state on for every radio. A radio can turn them on itself, but cannot turn them off when the group has turned them on
- `disabled` on the group cannot be overridden by a radio, exactly like `<fieldset disabled>`
- In a controlled group (`value` set) the group decides what is checked. `checked` on a radio is ignored, with a warning in development
- Radios keep submitting their own form values, so `FormData` is the same with and without the group

`PktInputWrapper` with `hasFieldset` and loose radios is still supported, but an "Other" text field belongs inside the group (below).

## An "Other" option with a text field

Put the text field inside the group, right after the "Other" option, and show it while "Other" is selected. The text field has its own `name` and is submitted as a separate field. It does not change the group's value or validation.

```jsx
const [transport, setTransport] = useState('')

<PktRadioGroup name="transport" label="…" value={transport} onValueChange={setTransport}>
  <PktRadioButton id="transport-a" value="a" label="A" />
  <PktRadioButton id="transport-other" value="other" label="Other" />
  {transport === 'other' && <PktTextinput id="transport-other-text" name="transport-other" label="What else?" />}
</PktRadioGroup>
```

```html
<pkt-radio-group id="transport" name="transport" label="…">
  <pkt-radiobutton id="transport-a" value="a" label="A"></pkt-radiobutton>
  <pkt-radiobutton id="transport-other" value="other" label="Other"></pkt-radiobutton>
  <pkt-textinput id="transport-other-text" name="transport-other" label="What else?" hidden disabled></pkt-textinput>
</pkt-radio-group>
<script>
  const group = document.getElementById('transport')
  const text = document.getElementById('transport-other-text')
  group.addEventListener('change', (event) => {
    if (event.target !== group) return
    text.hidden = text.disabled = group.value !== 'other'
  })
</script>
```

- In Elements, hide the field with `hidden` and `disabled` together, so it is not submitted while hidden. `hidden` works on Punkt elements because punkt-css sets `[hidden] { display: none !important }`
- In Elements, the text field's own events pass through the group like through a `fieldset`, so check `event.target === group` in a `change` listener on the group. In React, the group's `onChange` only fires for its options

## Examples

### React

```jsx
import { useState } from 'react'
import { PktRadioButton, PktRadioGroup } from '@oslokommune/punkt-react'

const [transport, setTransport] = useState('')

<PktRadioGroup
  name="transport"
  label="Travel method"
  value={transport}
  onValueChange={setTransport}
  layout="horizontal"
  hasTile
  required
>
  <PktRadioButton id="bus" value="bus" label="Bus" />
  <PktRadioButton id="train" value="train" label="Train" />
</PktRadioGroup>
```

### Elements

```html
<pkt-radio-group id="transport" name="transport" label="Travel method" value="train" required>
  <pkt-radiobutton id="bus" value="bus" label="Bus"></pkt-radiobutton>
  <pkt-radiobutton id="train" value="train" label="Train"></pkt-radiobutton>
</pkt-radio-group>

<script>
  document.getElementById('transport').addEventListener('change', (event) => {
    console.log(event.target.name, event.target.value)
  })
</script>
```
