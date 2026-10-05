# Checkbox Group

Checkbox Group wraps a set of checkboxes in a fieldset with label, help text and error message, and gives them a shared name, size and value. Switches (`isSwitch`) can be part of the group, also mixed with checkboxes.

**When you use the group, the group is the form field.** Put `name`, `value`, `onChange`, `onBlur`, `required` and form-library bindings on the group, not on the checkboxes. The group element has `name`, a readable and writable `value` and `focus()` (React: on `ref`; Elements: on the element), and its events carry the group as `event.target`.

## Availability

| Package        | Available | Tag / Import                                                                                        |
| -------------- | --------- | --------------------------------------------------------------------------------------------------- |
| React          | Yes       | `<PktCheckboxGroup>` — `import { PktCheckboxGroup } from '@oslokommune/punkt-react'`                |
| Elements       | Yes       | `<pkt-checkbox-group>` — `import '@oslokommune/punkt-elements/dist/pkt-checkbox-group.js'`          |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/19/elements/pkt-checkbox-group.js" type="module">` |

Dark mode: Yes

The Elements bundle registers `pkt-checkbox` too.

## Props / Attributes

| Prop (React)                  | Attribute (Elements)          | Type                                  | Default        | Description                                                                                                       |
| ----------------------------- | ----------------------------- | ------------------------------------- | -------------- | ----------------------------------------------------------------------------------------------------------------- |
| `label`                       | `label`                       | string                                | **(required)** | Group label, rendered as the `legend`                                                                             |
| `id`                          | `id`                          | string                                | —              | Group id. Also the fallback `name` for the checkboxes                                                             |
| `name`                        | `name`                        | string                                | —              | Given to checkboxes without their own `name`. Falls back to `id`, then to a generated name                        |
| `value`                       | `value`                       | string[]                              | —              | Values of the checked checkboxes in DOM order, like `FormData.getAll(name)`. Disabled checkboxes are left out. Elements also takes a comma-separated attribute |
| `defaultValue`                | —                             | string[]                              | —              | React only: initially checked values of an uncontrolled group. Elements uses the `value` attribute                |
| `layout`                      | `layout`                      | `"vertical"` \| `"horizontal"`        | `"vertical"`   | Direction of the checkboxes. The gap follows the size: 16, 12 or 8px                                              |
| `inputSize`                   | `input-size`                  | `"small"` \| `"medium"` \| `"xsmall"` | `"medium"`     | Size of the group. Checkboxes inherit it unless they set their own                                                |
| `hasTile`                     | `hasTile`                     | boolean                               | `false`        | Shows every checkbox as a tile                                                                                    |
| `required`                    | `required`                    | boolean                               | `false`        | At least one checkbox must be checked. Not passed on as `required` to the checkboxes                              |
| `disabled`                    | `disabled`                    | boolean                               | `false`        | Disables every checkbox, like `<fieldset disabled>`                                                               |
| `hasError`                    | `hasError`                    | boolean                               | `false`        | Error state on the group and every checkbox                                                                       |
| `errorMessage`                | `errorMessage`                | string                                | —              | Shown below the checkboxes when `hasError` is set                                                                 |
| `helptext`                    | `helptext`                    | string                                | —              | Help text above the checkboxes. Elements also takes a `helptext` slot                                             |
| `helptextDropdown`            | `helptextDropdown`            | string                                | —              | Expandable help text                                                                                              |
| `useWrapper`                  | `useWrapper`                  | boolean                               | `true`         | `false` hides label and help text visually; the group keeps its accessible name                                   |
| `requiredTag` / `optionalTag` | `requiredTag` / `optionalTag` | boolean                               | `false`        | Tags next to the label                                                                                            |
| `tagText`                     | `tagText`                     | string                                | —              | Custom tag next to the label                                                                                      |

## Events

| Event (React)   | Event (Elements) | Description                                                                                                  |
| --------------- | ---------------- | ------------------------------------------------------------------------------------------------------------ |
| `onChange`      | `change`         | Fired by the group when the user changes the selection. `event.target` is the group, `event.target.value` its value (`string[]`) |
| —               | `input`          | Fired by the group just before `change`, with the value already updated                                       |
| `onValueChange` | `value-change`   | The group's new value (`string[]`). React: a callback. Elements: `CustomEvent` with `detail`                     |
| `onFocus`       | `focus`          | Focus enters the group. Not fired when focus moves between checkboxes                                           |
| `onBlur`        | `blur`           | Focus leaves the group. Not fired when focus moves between checkboxes                                           |

In Elements the checkboxes' own events stop at the group, so a listener on the group or the form gets one event per change, with the group as target. Listeners on a checkbox itself still fire. In React the checkboxes' own `onChange` still runs. Setting `value` from code fires no events.

## Form libraries

React Hook Form works with every pattern: `register` on the group, `Controller` with `{...field}` spread on the group, `Controller` with `onValueChange={field.onChange}`, or `register` on each checkbox. Formik-style handlers that read `event.target.name` and `event.target.value` get the group's name and value.

```jsx
<PktCheckboxGroup label="…" {...register('rooms', { required: true })}>
  <PktCheckbox id="a" value="a" label="A" />
  <PktCheckbox id="b" value="b" label="B" />
</PktCheckboxGroup>

<Controller
  name="rooms"
  control={control}
  render={({ field }) => (
    <PktCheckboxGroup label="…" {...field}>
      <PktCheckbox id="a" value="a" label="A" />
    </PktCheckboxGroup>
  )}
/>
```

## Required

HTML has no "at least one" rule for checkboxes, so the group does what you would do by hand: while none is checked, every checkbox gets a custom validity message, "Velg minst ett alternativ" (`validation.checkboxGroupValueMissing` in the strings catalogue). The flag is `validity.customError`, not `valueMissing`, in both frameworks. On submit the browser focuses the first checkbox. A checkbox with its own `required` must still be checked.

## Rules for the checkboxes

- The group only gives defaults. `name`, `inputSize` and other props set on a checkbox win
- `hasTile` and `hasError` on the group turn the state on for every checkbox. A checkbox can turn them on itself, but cannot turn them off when the group has turned them on
- `disabled` on the group cannot be overridden by a checkbox, exactly like `<fieldset disabled>`
- In a controlled group (`value` set) the group decides what is checked. `checked` on a checkbox is ignored, with a warning in development
- Checkboxes keep submitting their own form values, so `FormData` is the same with and without the group

`PktInputWrapper` with `hasFieldset` and loose checkboxes is still supported, but an "Other" text field belongs inside the group (below).

## An "Other" option with a text field

Put the text field inside the group, right after the "Other" option, and show it while "Other" is selected. The text field has its own `name` and is submitted as a separate field. It does not change the group's value or validation.

```jsx
const [alerts, setAlerts] = useState([])

<PktCheckboxGroup name="alerts" label="…" value={alerts} onValueChange={setAlerts}>
  <PktCheckbox id="alerts-a" value="a" label="A" />
  <PktCheckbox id="alerts-other" value="other" label="Other" />
  {alerts.includes('other') && <PktTextinput id="alerts-other-text" name="alerts-other" label="What else?" />}
</PktCheckboxGroup>
```

```html
<pkt-checkbox-group id="alerts" name="alerts" label="…">
  <pkt-checkbox id="alerts-a" value="a" label="A"></pkt-checkbox>
  <pkt-checkbox id="alerts-other" value="other" label="Other"></pkt-checkbox>
  <pkt-textinput id="alerts-other-text" name="alerts-other" label="What else?" hidden disabled></pkt-textinput>
</pkt-checkbox-group>
<script>
  const group = document.getElementById('alerts')
  const text = document.getElementById('alerts-other-text')
  group.addEventListener('change', (event) => {
    if (event.target !== group) return
    text.hidden = text.disabled = !group.value.includes('other')
  })
</script>
```

- In Elements, hide the field with `hidden` and `disabled` together, so it is not submitted while hidden. `hidden` works on Punkt elements because punkt-css sets `[hidden] { display: none !important }`
- In Elements, the text field's own events pass through the group like through a `fieldset`, so check `event.target === group` in a `change` listener on the group. In React, the group's `onChange` only fires for its options

## Examples

### React

```jsx
import { useState } from 'react'
import { PktCheckbox, PktCheckboxGroup } from '@oslokommune/punkt-react'

const [rooms, setRooms] = useState([])

<PktCheckboxGroup name="rooms" label="Room type" value={rooms} onValueChange={setRooms} required>
  <PktCheckbox id="single" value="single" label="Single room" />
  <PktCheckbox id="double" value="double" label="Double room" />
  <PktCheckbox id="alerts" value="alerts" label="Send me alerts" isSwitch />
</PktCheckboxGroup>
```

### Elements

```html
<pkt-checkbox-group id="rooms" name="rooms" label="Room type" value="single,double" required>
  <pkt-checkbox id="single" value="single" label="Single room"></pkt-checkbox>
  <pkt-checkbox id="double" value="double" label="Double room"></pkt-checkbox>
</pkt-checkbox-group>
```
