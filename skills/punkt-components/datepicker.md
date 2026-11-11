# Datepicker

Datepicker lets the user select a date, multiple dates, or a date range. The user can either type a date directly in the field or pick from a visual calendar. Well suited for forms where the date matters for the next step.

## Availability

| Package        | Available | Tag / Import                                                                                   |
| -------------- | --------- | ---------------------------------------------------------------------------------------------- |
| React          | Yes       | `<PktDatepicker>` — `import { PktDatepicker } from '@oslokommune/punkt-react'`                 |
| Elements       | Yes       | `<pkt-datepicker>` — `import '@oslokommune/punkt-elements/dist/pkt-datepicker.js'`             |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/20/elements/pkt-datepicker.js" type="module">` |

Dark mode: Yes

**Note:** The React datepicker (`PktDatepicker`) and its internal calendar (`PktCalendar`) are pure React components — not Lit/Web Component wrappers. They do not depend on `@oslokommune/punkt-elements` at runtime. The React components use standard React props and callbacks (e.g. `onValueChange` with `string[]`) rather than CustomEvents.

## Variants

| Variant        | Use                                     |
| -------------- | --------------------------------------- |
| Single date    | Select one specific date                |
| Multiple dates | Select several dates at once            |
| Range          | Select a period with start and end date |

In range mode the value is read left to right, so a to-date needs a from-date. Emptying the to-field keeps the from-date. A to-date with no from-date stays in its field while the user is still in the datepicker, so they can type or pick a new from-date, and is cleared when focus leaves the datepicker. A picked from-date after the to-date starts a new range. The calendar only shows what is complete: nothing without a from-date, and the to-date once the range is complete.

## Usage guidelines

**Use datepicker when:**

- You want the user to select a date or date range
- There is a need for both free text and calendar input
- The date is significant for further steps in a form or process

**Avoid datepicker when:**

- The date is not significant for the task
- The user only needs to choose between a few specific dates — use Select or Radio Button instead

**Content guidelines:**

- Use label and help text to explain the format or rules (e.g. max number of dates, restricted choices)
- Describe what should be selected — avoid unclear labels

## Props / Attributes

| Prop (React)             | Attribute (Elements)     | Type                                  | Default        | Description                                                                                                     |
| ------------------------ | ------------------------ | ------------------------------------- | -------------- | --------------------------------------------------------------------------------------------------------------- |
| `name`                   | `name`                   | string                                | —              | Form field name                                                                                                 |
| `form`                   | `form`                   | string                                | —              | `id` of the `<form>` the field belongs to, when it sits outside it                                              |
| `id`                     | `id`                     | string                                | —              | Unique identifier                                                                                               |
| `label`                  | `label`                  | string                                | —              | Label displayed above the field                                                                                 |
| `helptext`               | `helptext`               | string                                | —              | Help text below the label                                                                                       |
| `helptextDropdown`       | `helptextDropdown`       | string                                | —              | Expandable help text content                                                                                    |
| `helptextDropdownButton` | `helptextDropdownButton` | string                                | —              | Deprecated. Use `strings`, see [Strings](strings.md)                                                            |
| `dateformat`             | `dateformat`             | string                                | `"dd.MM.yyyy"` | Date format string                                                                                              |
| `currentmonth`           | `currentmonth`           | string (ISO date)                     | —              | Month displayed in the calendar                                                                                 |
| `value`                  | `value`                  | string (ISO date)                     | —              | Selected value (comma-separated for multiple and range). React also accepts `string[]`                          |
| `excludeweekdays`        | `excludeweekdays`        | `string[]`                            | —              | Weekdays to exclude (1-7). Elements takes a comma-separated attribute                                           |
| `excludedates`           | `excludedates`           | `string[]` (ISO dates)                | —              | ISO dates to exclude. Elements takes a comma-separated attribute                                                |
| `min`                    | `min`                    | string (ISO date)                     | —              | Earliest allowed date                                                                                           |
| `max`                    | `max`                    | string (ISO date)                     | —              | Latest allowed date                                                                                             |
| `weeknumbers`            | `weeknumbers`            | boolean                               | `false`        | Show week numbers                                                                                               |
| `withcontrols`           | `withcontrols`           | boolean                               | `false`        | Show month/year selectors                                                                                       |
| `multiple`               | `multiple`               | boolean                               | `false`        | Allow selecting multiple dates                                                                                  |
| `maxlength`              | `maxlength`              | number                                | —              | Max number of dates in multi select. `0` means no limit                                                        |
| `range`                  | `range`                  | boolean                               | `false`        | Enable date range selection (from–to)                                                                           |
| `showRangeLabels`        | `show-range-labels`      | boolean                               | `false`        | Show from/to labels in range mode                                                                               |
| `today`                  | `today`                  | string (ISO date)                     | —              | Override "today" in the calendar (useful for snapshot tests)                                                    |
| `calendarOpen`           | `calendar-open`          | boolean                               | `false`        | Calendar popup is open                                                                                          |
| `hasError`               | `hasError`               | boolean                               | `false`        | Shows error state                                                                                               |
| `errorMessage`           | `errorMessage`           | string                                | —              | Error message shown below the field                                                                             |
| `disabled`               | `disabled`               | boolean                               | `false`        | Disables the datepicker                                                                                         |
| `fullwidth`              | `fullwidth`              | boolean                               | `false`        | Field takes full width                                                                                          |
| `inputSize`              | `input-size`             | `"xsmall"` \| `"small"` \| `"medium"` | `"medium"`     | Field height                                                                                                    |
| `required`               | `required`               | boolean                               | `false`        | Field is required                                                                                               |
| `requiredTag`            | `requiredTag`            | boolean                               | `false`        | Show "Required" tag next to label                                                                               |
| `requiredText`           | `requiredText`           | string                                | —              | Deprecated. Use `strings`, see [Strings](strings.md)                                                            |
| `optionalTag`            | `optionalTag`            | boolean                               | `false`        | Show "Optional" tag next to label                                                                               |
| `optionalText`           | `optionalText`           | string                                | —              | Deprecated. Use `strings`, see [Strings](strings.md)                                                            |
| `tagText`                | `tagText`                | string                                | —              | Custom tag text next to label                                                                                   |
| `useWrapper`             | `useWrapper`             | boolean                               | `true`         | Show label, help text, and wrapper                                                                              |
| `inline`                 | `inline`                 | boolean                               | `false`        | Field flows with the surrounding text                                                                           |
| `hasFieldset`            | `hasFieldset`            | boolean                               | derived        | Use fieldset and legend instead of label                                                                        |
| `counter`                | `counter`                | boolean                               | derived        | Show the date counter. When unset, it shows with `multiple` and a `maxlength` above 0. `false` hides it |
| `ariaDescribedby`        | `aria-describedby`       | string                                | —              | ID of an element that describes the field                                                                       |

## Forms

The two frameworks submit multiple dates differently:

- **Elements** works like `<select>`. One date sends one entry. With `multiple` or `range` it sends one entry per date, like `<select multiple>`, pre-filled values included: `dato=2024-06-10&dato=2024-06-20`.
- **React** sends all dates as one comma-separated entry: `dato=2024-06-10,2024-06-20`. Use `onValueChange` when you need the dates as a `string[]`.

Without `name`, both use the `id` as the field name.

## Events

| Event (React)   | Event (Elements) | Description                                                                                                                                                                         |
| --------------- | ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `onChange`      | `change`         | Fires when a date is selected. `e.target.value` contains comma-joined ISO date string. With `range`: fires when the range is complete or emptied, not when only the from-date is set |
| `onValueChange` | `value-change`   | Fires on every value change, also when only the from-date of a range is set. React: called with a `string[]` of ISO dates. Elements: `event.detail` is an ISO date string in single mode and an array with `multiple` or `range` |
| `onFocus`       | `focus`          | Fires once when focus enters the datepicker: the field, the calendar button or the calendar. Moving between them fires nothing                                                      |
| `onBlur`        | `blur`           | Fires once when focus leaves the datepicker, also from the calendar. React: `e.target` is the named hidden input, as with `onChange`, so `register()` from react-hook-form works    |
| —               | —                | `toggleHelpText` is dispatched by the inner input wrapper with `bubbles: false`. `pkt-datepicker` does not re-dispatch it, and React `PktDatepicker` has no `onToggleHelpText` prop |

## Accessibility

- Label must always be visible — if hidden visually, use `pkt-sr-only` for screen reader access
- The datepicker must work without a mouse: open calendar with keyboard, navigate with arrow keys, select with Enter, close with Esc
- Screen readers must be able to read the selected date — use proper ARIA roles and attributes
- Show help text explaining the date format (e.g. `dd.MM.yyyy`) and any restrictions — don't rely on placeholder alone
- If the field has an error, the error message must be linked to the input with `aria-describedby`

## Examples

### React

```jsx
import { PktDatepicker } from '@oslokommune/punkt-react'

{
  /* Single date picker */
}
;<PktDatepicker
  label="Date of birth"
  name="birthdate"
  id="birthdate"
  helptext="Format: dd.mm.yyyy"
  requiredTag
  required
/>

{
  /* Date range */
}
;<PktDatepicker
  label="Travel period"
  name="travel-period"
  id="travel-period"
  range
  min="2024-01-01"
  max="2024-12-31"
/>

{
  /* Multiple dates with week numbers */
}
;<PktDatepicker
  label="Select dates"
  name="dates"
  id="dates-multi"
  multiple
  maxlength={5}
  weeknumbers
  helptext="Select up to 5 dates"
/>
```

### Elements

```html
<pkt-datepicker
  label="Date of birth"
  name="birthdate"
  id="birthdate"
  helptext="Format: dd.mm.yyyy"
  requiredTag
  required
></pkt-datepicker>

<pkt-datepicker
  label="Travel period"
  name="travel-period"
  id="travel-period"
  range
  min="2024-01-01"
  max="2024-12-31"
></pkt-datepicker>
```
