# Form Integration

Form input components are the most complex part of the elements package. They use the browser's `ElementInternals` API (with polyfill) to participate natively in HTML `<form>` elements — enabling validation, form submission, and reset.

## Class hierarchy for inputs

```
PktElementWithSlot (light DOM, slot content)
  └─ PktInputElement (form association, validation, events)
      └─ PktOptionsInputElement (option management for selects/comboboxes)
```

## PktInputElement

**File:** `src/base-elements/input-element.ts`
**Extends:** `PktElementWithSlot`

### Form association

Every input element is a form-associated custom element:

```typescript
export class PktInputElement<T = {}, TOption extends PktInputOption = PktInputOption>
  extends PktElement<Props & T> {

  static get formAssociated() {
    return true
  }

  constructor() {
    super()
    this.internals = this.attachInternals()
  }
}
```

The `internals` object (`ElementInternals`) provides:
- `setFormValue()` — set the value submitted with the form
- `setValidity()` — set custom validity state
- `reportValidity()` — trigger browser validation UI
- `form` — reference to the parent `<form>`
- `states` — `CustomStateSet` for CSS `:state()` selectors

### The `form` attribute

Because the host element is the form-associated element, the browser resolves the `form` content
attribute natively — `<pkt-textinput form="my-form">` submits with that form even when it sits
outside it. Nothing needs forwarding to the inner input, which carries `form=""` on purpose so it
never submits alongside the host.

Use the `form` getter to find the owning form. It follows the HTML form owner rules: the `form`
attribute when set (`form=""` means no owner), otherwise the closest `<form>` ancestor.

```typescript
get form(): HTMLFormElement | null
```

**Never use `this.closest('form')` to find the owning form** — it misses controls linked with the
`form` attribute. The same applies to scoping a query to the form: `form.querySelectorAll()` only
finds descendants, so query the root node and filter on `.form` instead (see
`radioGroupMembers()`).

Components that are not form-associated (`pkt-fileupload`) declare `form` as a plain string
property and forward it to the native inputs they render.

### Properties

PktInputElement declares a large set of properties that all input subclasses inherit:

**Standard form properties:**
```typescript
@property({ type: String, reflect: true }) name: string = ''
@property({ type: String, reflect: true }) id: string = uuidish()
@property({ type: Boolean, reflect: true }) disabled: boolean = false
@property({ type: Boolean, reflect: true }) readonly: boolean = false
@property({ type: Boolean, reflect: true }) required: boolean = false
@property({ type: String, reflect: true }) placeholder: string | null = null
@property({ type: String, reflect: true }) pattern: string | null = null
@property() defaultValue: string | string[] | null = null
```

**Validation properties:**
```typescript
@property({ type: Number, reflect: true }) maxlength: number | null = null
@property({ type: Number, reflect: true }) minlength: number | null = null
@property({ reflect: true }) max: string | number | null = null  // Custom converter
@property({ reflect: true }) min: string | number | null = null  // Custom converter
@property({ type: Number, reflect: true }) step: number | null = null
```

**Checkable properties (checkbox, radio):**
```typescript
@property({ type: Boolean, reflect: true }) checked?: boolean | string | null
@property({ type: Boolean }) defaultChecked?: boolean
```

**Multi-value properties:**
```typescript
@property({ type: Boolean }) multiple?: boolean
@property({ type: Boolean }) range?: boolean
```

**InputWrapper integration properties:**
```typescript
@property({ type: String }) label: string | null = null
@property({ type: String }) helptext: string = ''
@property({ type: String }) errorMessage: string = ''
@property({ type: Boolean }) hasError: boolean = false
@property({ type: Boolean }) counter: boolean = false
@property({ type: Boolean }) fullwidth: boolean = false
@property({ type: Boolean }) inline: boolean = false
@property({ converter: booleanishConverter }) useWrapper: Booleanish = true
@property({ type: Boolean }) optionalTag: boolean = false
@property({ type: Boolean }) requiredTag: boolean = false
// ... and more
```

**State:**
```typescript
@state() touched: boolean = false
```

**Controller declarations:**
```typescript
declare optionsController?: PktOptionsSlotController
```

### Key concept: value vs _value

Subclasses declare their own `value` property (not declared in the base class to allow both property and accessor patterns):

```typescript
// Most components: simple property
@property({ type: String, reflect: true }) value: string = ''

// Complex components (datepicker): getter/setter
@property() get value() { /* ... */ }
set value(v) { /* ... */ }
```

The base class also uses `_value` as an internal array representation for multi-value inputs.

### Core methods

#### `onChange(value)` — Main form update handler

Call this when the input's value changes (e.g., user types, selects an option). It handles:

1. Marks input as `touched` on first interaction
2. Normalizes value (handles comma-separated strings for multi-select)
3. Updates form value via `setFormValue()`
4. Validates via `manageValidity()`
5. Dispatches `change` and `value-change` events

```typescript
protected onChange(value: string | string[]): void {
  if (!this.touched) {
    this.touched = true
    if (value) this.setFormValue(value)
    return
  }
  const normalizedValue = this.normalizeValue(value)
  this.setFormValue(normalizedValue)
  this.validate()
  this.dispatchChangeEvents(normalizedValue)
}
```

#### `valueChanged(value, old)` — Attribute/property change handler

Called when the `value` property/attribute changes (programmatic or external). It:

1. Handles string → array conversion for multi/range values
2. Updates both `value` and `_value`
3. Calls `clearInputValue()` when value becomes empty
4. Calls `onChange()` when value actually changes

Subclasses can override this for custom transformation:
```typescript
protected valueChanged(value: string | string[] | null, _old: string | string[] | null): void {
  // Custom transformation
  super.valueChanged(transformedValue, _old)
}
```

#### `valueChecked(value)` — Checkbox/radio handler

For checkable inputs only, called on user interaction. Updates checked state and form value:

1. Coordinates radio groups (unchecks other radios with same name and form owner)
2. Updates `checked`, then calls `syncCheckedState()`: `ariaChecked`, form value (`value` string when checked, `'on'` without a value, `null` when unchecked), `CustomStates` (`--checked`) and the inner `<input>`
3. Revalidates (the whole name group for radios)
4. Dispatches change events

It does **not** call `reportValidity()`: like native inputs, validity updates on change and is reported on submit.

```typescript
protected valueChecked(value: string | boolean | null): void
```

**Do not use `valueChanged` or `onChange` for checkboxes and radios** — use `valueChecked` instead.

Setting `checked`, `value`, `required` or `name` from code is picked up in `updated()`, which runs `syncCheckedState()` and revalidates **without dispatching events** — the same as setting `checked` on a native input.

#### Native parity for checkable inputs

Checkbox and radio follow plain HTML. Keep it that way; `src/base-elements/checkable-native-parity.test.ts` and `packages/e2e/tests/native-parity.spec.ts` (which compares against plain HTML on the same page) guard it.

- `required` on a checkbox: `valueMissing` while unchecked
- `required` on a radio makes the whole name group (same `name` and form owner) required; the requirement is met when any radio in the group is checked, and every radio in the group reports the same validity
- `change` fires only on the radio that becomes checked, never on the one that is unchecked
- Form reset restores the default checked state (`defaultChecked`, or `checked` in the markup) and fires no events
- A radio without `name` gets `name = id`, i.e. its own group. This fallback is deliberate: without a name there is no way to know what belongs together
- The inner `<input type="radio">` has `form=""`, so it has no form owner and would share a native group with every inner radio of the same name on the page. Its name is therefore `internalName`: `{name}-internal` outside a form, `{name}-internal-{form key}` with a form owner (the form's `id`, or a generated key). Radios linked with the `form` attribute get the same key as the radios inside that form, so arrow keys and unchecking stay within the real group. `formAssociatedCallback()` re-renders when the form owner changes

#### Option groups (`pkt-radio-group`, `pkt-checkbox-group`)

The groups extend `PktOptionGroupElement` (`src/base-elements/option-group-element.ts`), not `PktInputElement`: they are not form-associated, and the options keep submitting their own values. The group renders `pkt-input-wrapper` with `hasFieldset`, forwards its default and `helptext` slots to it, and provides a context (`radio-group-context.ts`, `checkbox-group-context.ts`) that the options consume as `groupContext`.

- **Name:** the base class reads `this.optionGroup?.name` in `willUpdate()`. An option whose `name` is empty or came from a fallback (`name = id`, or an earlier group name) takes the group's name. A `name` set by the user always wins
- **Booleans:** `hasTile` and `hasError` from the group are OR-ed with the option's own value. Elements cannot tell "not set" from `false`, so the group can only turn them on
- **Required:** a radio group passes `required` through the context, and `isRequired` includes it in the name-group check. A checkbox group never passes `required`; it calls `setGroupValidity(message)` on every option instead, which `manageValidity()` adds as `customError` (together with `valueMissing` when the option is required itself)
- **Disabled:** the group passes `disabled` to the wrapper, which sets `disabled` on the fieldset. The options pick it up through `formDisabledCallback()`
- **The group is the form field.** `value` is an accessor that reads the options' checked state, so it is always current. Setting it stores the value and checks the matching options without events; options added later get the stored value. `focus()` focuses the checked (or first) option's inner input
- **Events:** the group listens in its constructor (before any consumer listener) for `change`, `input`, `value-change`, `focus` and `blur` from its options, stops them with `stopImmediatePropagation()` and fires its own with the group as target. `focus`/`blur` come from native `focusin`/`focusout` and only fire when focus enters or leaves the group. Listeners on an option itself still get the option's events
- **Event order on the options:** `valueChecked()` dispatches `input`, `change` and `value-change` after the checked state is updated, like native inputs. The inner input's own `input` event is stopped
- **Tests:** `src/tests/option-contexts.ts` runs the same option test alone and in a group. Use it for any behaviour the option must keep in both places

#### `setFormValue(value)` — Updates the form value

Handles both single values and arrays:

```typescript
protected setFormValue(value: string | string[]): void {
  if (Array.isArray(value)) {
    const form = new FormData()
    value.forEach((v) => form.append(this.name, v))
    this.writeFormValue(form)
  } else {
    this.writeFormValue(value)
  }
}
```

**Never call `this.internals.setFormValue()` directly**, in the base class or a subclass. Use `setFormValue()` (or `writeFormValue()` inside the base class). With `element-internals-polyfill` (jsdom, Safari before 16.4) the form value is a hidden `<input>` next to the host, and the polyfill removes it whenever the host is disconnected, including when the slot system moves the host into `.pkt-inputwrapper__options` or a group. `writeFormValue()` remembers the last value when the polyfill is active, and `connectedCallback()` writes it again in a microtask, after the polyfill has cleaned up. `src/base-elements/polyfill-form-value.test.ts` guards it.

#### `manageValidity(input)` — Validation

Validates the input and sets the `ElementInternals` validity state. Checks, in order:
1. `required` + empty value → `valueMissing` (checkbox: unchecked; radio: no radio in the name group checked)
2. `typeMismatch` / `badInput` → `typeMismatch`
3. `patternMismatch` → `patternMismatch`
4. `tooShort` / `minlength` → `tooShort`
5. `tooLong` / `maxlength` → `tooLong`
6. `rangeUnderflow` → `rangeUnderflow` (with `{min}` substitution)
7. `stepMismatch` → `stepMismatch`
8. `rangeOverflow` → `rangeOverflow` (with `{max}` substitution)
9. `customError` → uses the input's `validationMessage`
10. Otherwise → valid (`setValidity({})`)

Error messages come from `this.pktStrings.get('validation')` in the shared string catalogue.

#### `dispatchChangeEvents(value)` — Event dispatching

Dispatches both standard and custom events:

```typescript
protected dispatchChangeEvents(value: unknown): void {
  this.dispatchEvent(new Event('change', { bubbles: true, composed: true }))
  this.dispatchEvent(new CustomEvent('value-change', {
    detail: value,
    bubbles: true,
    composed: true,
  }))
}
```

#### `onFocus()`, `onBlur()`, `onInput()`

Dispatch corresponding native events. Call these from event handlers in the render method.

#### `formResetCallback()`

Called automatically by the browser when the parent `<form>` is reset. Resets:
- `touched` state
- Options (clears `selected`)
- Checkbox/radio (back to the default checked state, no events)
- Value (reverts to `defaultValue`)
- Validity

#### `formDisabledCallback()` and `isDisabled`

The browser calls `formDisabledCallback(disabled)` when the element is disabled by an ancestor `<fieldset disabled>` (or its own `disabled` attribute). The base class stores it, and `isDisabled` combines it with the `disabled` property.

**Render and guard interaction from `this.isDisabled`, never `this.disabled`.** Native inner inputs are disabled by the fieldset anyway, but Punkt's own state is not: wrapper and label classes, tile classes, and non-native controls like the select-only combobox (a `div` with `tabindex`) would otherwise stay active inside a disabled fieldset. Never assign to `disabled` from the callback: it is reflected, and the host's own `disabled` attribute would keep it disabled when the fieldset is enabled again.

#### `firstUpdated()`

The base class `firstUpdated` does important setup:
- Sets `defaultValue` from initial `value`
- Handles `defaultChecked`
- Sets ARIA attributes (`required`, `disabled`)
- Sets initial form value
- Runs initial validation

### Building an input component

Here's the pattern for a simple text input:

```typescript
import { html } from 'lit'
import { customElement, property } from 'lit/decorators.js'
import { Ref, createRef, ref } from 'lit/directives/ref.js'
import { ifDefined } from 'lit/directives/if-defined.js'
import { PktInputElement } from '@/base-elements/input-element'
import { forwardSlots } from '@/directives/slot-content'
import '@/components/input-wrapper'

export class PktTextinput extends PktInputElement<Props> {
  inputRef: Ref<HTMLInputElement> = createRef()

  @property({ type: String, reflect: true }) value: string = ''

  attributeChangedCallback(name: string, _old: string, value: string): void {
    if (name === 'value' && this.value !== _old) {
      this.valueChanged(value, _old)
    }
    super.attributeChangedCallback(name, _old, value)
  }

  render() {
    return html`
      <pkt-input-wrapper
        ${forwardSlots(this, ['helptext'])}
        ?disabled=${this.isDisabled}
        ?hasError=${this.hasError}
        ?required=${this.required}
        label=${ifDefined(this.label)}
        errorMessage=${ifDefined(this.errorMessage)}
        helptext=${ifDefined(this.helptext)}
        forId=${this.id + '-input'}
      >
        <input
          ${ref(this.inputRef)}
          type="text"
          class="pkt-input"
          id=${this.id + '-input'}
          name=${this.name || this.id}
          value=${this.value}
          ?disabled=${this.isDisabled}
          ?readonly=${this.readonly}
          ?required=${this.required}
          placeholder=${ifDefined(this.placeholder)}
          @change=${(e: Event) => {
            this.touched = true
            this.value = (e.target as HTMLInputElement).value
            e.stopImmediatePropagation()
          }}
          @input=${(e: InputEvent) => {
            this.onInput()
            e.stopImmediatePropagation()
          }}
          @focus=${(e: FocusEvent) => {
            this.onFocus()
            e.stopImmediatePropagation()
          }}
          @blur=${(e: FocusEvent) => {
            this.onBlur()
            e.stopImmediatePropagation()
          }}
        />
      </pkt-input-wrapper>
    `
  }
}
```

Key patterns:
- **`inputRef`** — Ref to the native `<input>` for validation
- **`e.stopImmediatePropagation()`** — prevents duplicate events bubbling from both native input and custom element
- **`pkt-input-wrapper`** — wraps the input with label, helptext, error display
- **`forwardSlots(this, ['helptext'])`** — hands slotted helptext to the wrapper without a holder element (see [Light DOM & Slots](light-dom-and-slots.md))
- **`this.isDisabled`** — follows both the `disabled` property and a disabled ancestor `<fieldset>`
- **`forId`** — associates wrapper label with the input's ID

### Event handling in input renders

All native input events (`change`, `input`, `focus`, `blur`) must:
1. Update the component state (e.g., `this.value = ...`)
2. Call the corresponding base class method (`this.onInput()`, `this.onFocus()`, etc.)
3. Stop immediate propagation to prevent duplicate events

```typescript
@change=${(e: Event) => {
  this.touched = true
  this.value = (e.target as HTMLInputElement).value
  e.stopImmediatePropagation()
}}
```

## PktOptionsInputElement

**File:** `src/base-elements/options-input-element.ts`
**Extends:** `PktInputElement`

Adds option management for select-like components.

### Properties

```typescript
@property({ type: Array, attribute: 'options' }) protected _optionsProp: TOption[] = []
@state() _options: TOption[] = []
declare optionsController: PktOptionsSlotController
```

### Public API

```typescript
// Getter — returns options with computed selected state
public get options(): TOption[] {
  return this._options.map((option) => ({
    ...option,
    selected: this.isOptionSelected(option),
  }))
}

// Setter — updates props and triggers re-render
public set options(value: TOption[]) {
  this._optionsProp = value
  this.requestUpdate('_optionsProp', this._options)
}
```

### Key methods

#### `parseOptions()`
Resolves options from props or slot controller. Props take priority.

```typescript
protected parseOptions(): void {
  if (this._optionsProp.length > 0) {
    this._options = this._optionsProp
  } else if (this.optionsController?.nodes?.length > 0) {
    this._options = this.optionsController.options as TOption[]
  }
}
```

Called automatically in `willUpdate()` and should also be called in `connectedCallback()`.

#### `isOptionSelected(option)`
Determines if an option is selected based on `this.value`. Subclasses can override for custom logic.

#### `findOptionByValue(value)` / `getSelectedOptions()`
Helper methods for option lookup.

### Building an options component

```typescript
export class PktSelect extends PktOptionsInputElement<{}, TSelectOption> {
  inputRef: Ref<HTMLSelectElement> = createRef()

  @property({ type: String }) value: string = ''

  constructor() {
    super()
    this.optionsController = new PktOptionsSlotController(this)
  }

  connectedCallback(): void {
    super.connectedCallback()
    this.parseOptions()

    // Set initial value from selected option
    this._options.forEach((option) => {
      if (option.selected && !this.value) {
        this.value = option.value
      }
    })
  }

  render() {
    return html`
      <pkt-input-wrapper ...>
        <select ${ref(this.inputRef)} ...>
          ${this._options.map((option) => html`
            <option
              value=${option.value}
              ?selected=${this.value == option.value || option.selected}
              ?disabled=${option.disabled}
              ?hidden=${option.hidden}
            >${option.label}</option>
          `)}
        </select>
      </pkt-input-wrapper>
    `
  }
}

try {
  customElement('pkt-select')(PktSelect)
} catch (e) {
  console.warn('Forsøker å definere <pkt-select>, men den er allerede definert')
}
```

`slotContent` skips `<option>` and `<data>` children automatically, so the options never leak into a named slot.

Consumer can provide options as props or as children:

```html
<!-- Via props (JavaScript) -->
<pkt-select .options=${[{value: 'a', label: 'Option A'}, ...]}></pkt-select>

<!-- Via children (HTML) -->
<pkt-select>
  <option value="a">Option A</option>
  <option value="b" selected>Option B</option>
</pkt-select>
```

### PktInputOption interface

```typescript
export interface PktInputOption {
  value: string
  label?: string
  selected?: boolean
  disabled?: boolean
  hidden?: boolean
}
```

Components can extend this with additional fields (e.g., `TSelectOption`, `IPktComboboxOption`).
