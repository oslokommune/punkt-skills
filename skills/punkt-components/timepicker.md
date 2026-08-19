# Timepicker

Timepicker lets the user select a time of day. The user can type hours and minutes directly or pick from a scrollable dropdown. Well suited for forms where the time matters for the next step.

## Availability

| Package        | Available | Tag / Import                                                                                       |
| -------------- | --------- | -------------------------------------------------------------------------------------------------- |
| React          | Yes       | `<PktTimepicker>` — `import { PktTimepicker } from '@oslokommune/punkt-react'`                     |
| Elements       | Yes       | `<pkt-timepicker>` — `import '@oslokommune/punkt-elements/dist/pkt-timepicker.js'`                 |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/latest/elements/pkt-timepicker.js" type="module">` |

Dark mode: Yes

**Note:** The React timepicker (`PktTimepicker`) is a pure React component — not a Lit/Web Component wrapper. It does not depend on `@oslokommune/punkt-elements` at runtime. It uses standard React props and callbacks (e.g. `onChange`, `onValueChange`).

## Variants

| Variant                     | Use                                                           |
| --------------------------- | ------------------------------------------------------------- |
| Standard                    | Text fields for hours and minutes with a clock button that opens a scrollable dropdown |
| Hide picker (`hidePicker`)  | Only the text fields and a static clock icon — no dropdown    |
| Step arrows (`stepArrows`)  | Previous/next buttons on each side to step through times      |

## Usage guidelines

**Use timepicker when:**

- The user needs to enter a time as part of a form
- The time matters for the next step in a process
- You want to offer both free-text input and a visual picker
- The user needs to choose within a constrained time range (min/max)

**Avoid timepicker when:**

- The user only needs to choose between a few fixed times — use Select or Radio Button instead
- The time is not significant for the task

**Content guidelines:**

- Use label and help text to explain the expected format and any restrictions (e.g. time range, step interval)
- Describe what should be selected — avoid unclear labels like "Tid"

## Props / Attributes

| Prop (React)             | Attribute (Elements)       | Type               | Default          | Description                                              |
| ------------------------ | -------------------------- | ------------------ | ---------------- | -------------------------------------------------------- |
| `id`                     | `id`                       | string             | —                | Unique identifier (required)                             |
| `name`                   | `name`                     | string             | —                | Form field name; falls back to `id`                      |
| `label`                  | `label`                    | string             | —                | Label displayed above the field (required)               |
| `value`                  | `value`                    | string             | —                | Controlled value in `HH:MM` format                       |
| `defaultValue`           | —                          | string             | —                | Initial value (uncontrolled mode, React only)            |
| `min`                    | `min`                      | string             | —                | Earliest valid time (`HH:MM`)                            |
| `max`                    | `max`                      | string             | —                | Latest valid time (`HH:MM`)                              |
| `step`                   | `step`                     | number             | `60`             | Step in seconds. Must divide evenly into minutes or be exactly `3600` |
| `hidePicker`             | `hide-picker`              | boolean            | `false`          | Hides the clock button; shows a static icon              |
| `stepArrows`             | `step-arrows`              | boolean            | `false`          | Shows prev/next step buttons instead of the dropdown     |
| `helptext`               | `helptext`                 | string             | —                | Help text below the label                                |
| `helptextDropdown`       | `helptext-dropdown`        | string             | —                | Expandable help text content                             |
| `helptextDropdownButton` | `helptext-dropdown-button` | string             | `"Les mer"`      | Button text for expandable help                          |
| `disabled`               | `disabled`                 | boolean            | `false`          | Disables the timepicker                                  |
| `required`               | `required`                 | boolean            | `false`          | Field is required                                        |
| `fullwidth`              | `fullwidth`                | boolean            | `false`          | Field takes full width                                   |
| `hasError`               | `hasError`                 | boolean            | `false`          | Shows error state                                        |
| `errorMessage`           | `errorMessage`             | string             | —                | Error message shown below the field                      |
| `requiredTag`            | `requiredTag`              | boolean            | `false`          | Show "Required" tag next to label                        |
| `requiredText`           | `requiredText`             | string             | `"Må fylles ut"` | Text in the required tag                                 |
| `optionalTag`            | `optionalTag`              | boolean            | `false`          | Show "Optional" tag next to label                        |
| `optionalText`           | `optionalText`             | string             | `"Valgfritt"`    | Text in the optional tag                                 |
| `tagText`                | `tagText`                  | string             | —                | Custom tag text next to label                            |
| `useWrapper`             | `useWrapper`               | boolean            | `true`           | Show label, help text, and wrapper                       |
| `inline`                 | `inline`                   | boolean            | `false`          | Inline layout                                            |
| `strings`                | —                          | ITimepickerStrings | —                | Override labels for hours, minutes, buttons (React only) |
| `ariaDescribedby`        | `aria-describedby`         | string             | —                | ID of element describing the field                       |

## Events

| Event (React)      | Event (Elements) | Description                                                                             |
| ------------------ | ---------------- | --------------------------------------------------------------------------------------- |
| `onChange`          | `change`         | Fires when a time is committed. `e.target.value` contains the `HH:MM` string.          |
| `onValueChange`    | `value-change`   | Returns the committed `HH:MM` string (`string` in React, `CustomEvent.detail` in Elements) |
| `onInput`          | `input`          | Fires on each keystroke or arrow-key spin while editing                                 |
| `onFocus`          | `focus`          | Fires when focus enters the component from outside                                      |
| `onBlur`           | `blur`           | Fires when focus leaves the component entirely                                          |
| —                  | `toggleHelpText` | Fires when expandable help text opens/closes. `{ isOpen: boolean }`                     |

## Accessibility

- Label must always be visible — if hidden visually, use `pkt-sr-only` for screen reader access
- The hour and minute fields use `role="spinbutton"` with `aria-valuemin`, `aria-valuemax`, `aria-valuenow`, and `aria-valuetext`
- Each spinbutton's `aria-label` includes the component label, so VoiceOver announces e.g. "15, Timer, Møtetidspunkt" — matching native `<input type="time">` behavior
- The timepicker must work without a mouse: type digits directly, navigate between hour/minute with Arrow Left/Right, adjust values with Arrow Up/Down, open the dropdown with the clock button, navigate options with arrow keys, select with Enter, close with Esc
- If the field has an error, the error message is linked via `aria-describedby`
- The popup clock button has `aria-haspopup="listbox"` and `aria-expanded`

## Examples

### React

```jsx
import { PktTimepicker } from '@oslokommune/punkt-react'

{/* Basic timepicker */}
<PktTimepicker
  label="Møtetidspunkt"
  id="meeting-time"
  name="meeting-time"
  requiredTag
  required
/>

{/* With min, max, and step (5-minute intervals between 08:00–16:00) */}
<PktTimepicker
  label="Arbeidstid"
  id="work-time"
  name="work-time"
  min="08:00"
  max="16:00"
  step={300}
  helptext="Velg tidspunkt i 5-minutters intervaller"
/>

{/* Step arrows variant */}
<PktTimepicker
  label="Starttidspunkt"
  id="start-time"
  name="start-time"
  stepArrows
/>

{/* Hide picker variant */}
<PktTimepicker
  label="Påminnelse"
  id="reminder-time"
  name="reminder-time"
  hidePicker
/>

{/* Controlled with onChange */}
<PktTimepicker
  label="Tidspunkt"
  id="controlled-time"
  value={time}
  onChange={(e) => setTime(e.target.value)}
/>
```

### Elements

```html
<pkt-timepicker
  label="Møtetidspunkt"
  id="meeting-time"
  name="meeting-time"
  required-tag
  required
></pkt-timepicker>

<pkt-timepicker
  label="Arbeidstid"
  id="work-time"
  name="work-time"
  min="08:00"
  max="16:00"
  step="300"
  helptext="Velg tidspunkt i 5-minutters intervaller"
></pkt-timepicker>

<pkt-timepicker
  label="Starttidspunkt"
  id="start-time"
  name="start-time"
  step-arrows
></pkt-timepicker>

<pkt-timepicker
  label="Påminnelse"
  id="reminder-time"
  name="reminder-time"
  hide-picker
></pkt-timepicker>
```
