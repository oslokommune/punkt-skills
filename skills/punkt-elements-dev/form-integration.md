# Form Integration

Form input components are the most complex part of the elements package. They use the browser's `ElementInternals` API (with polyfill) to participate natively in HTML `<form>` elements — enabling validation, form submission, and reset.

## Class hierarchy for inputs

```
PktElement (light DOM)
  └─ PktInputElement (form association, validation, events)
      └─ PktOptionsInputElement (option management for selects/comboboxes)
```

## PktInputElement

**File:** `src/base-elements/input-element.ts`
**Extends:** `PktElement`

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
declare slotController?: PktSlotController
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

For checkable inputs only. Updates checked state and form value:

1. Coordinates radio groups (unchecks other radios with same name)
2. Updates `checked` property and `ariaChecked`
3. Sets form value (`value` string when checked, `null` when unchecked)
4. Manages `CustomStates` (`--checked`)
5. Dispatches change events

```typescript
protected valueChecked(value: string | boolean | null): void
```

**Do not use `valueChanged` or `onChange` for checkboxes and radios** — use `valueChecked` instead.

#### `setFormValue(value)` — Updates the form value

Handles both single values and arrays:

```typescript
protected setFormValue(value: string | string[]): void {
  if (Array.isArray(value)) {
    const form = new FormData()
    value.forEach((v) => form.append(this.name, v))
    this.internals.setFormValue(form)
  } else {
    this.internals.setFormValue(value)
  }
}
```

#### `manageValidity(input)` — Validation

Validates the input and sets the `ElementInternals` validity state. Checks, in order:
1. `required` + empty value → `valueMissing`
2. `typeMismatch` / `badInput` → `typeMismatch`
3. `patternMismatch` → `patternMismatch`
4. `tooShort` / `minlength` → `tooShort`
5. `tooLong` / `maxlength` → `tooLong`
6. `rangeUnderflow` → `rangeUnderflow` (with `{min}` substitution)
7. `stepMismatch` → `stepMismatch`
8. `rangeOverflow` → `rangeOverflow` (with `{max}` substitution)
9. `customError` → uses the input's `validationMessage`
10. Otherwise → valid (`setValidity({})`)

Error messages come from `strings.forms.messages` (Norwegian translations).

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
- Checkbox/radio (unchecks)
- Value (reverts to `defaultValue`)
- Validity

#### `firstUpdated()`

The base class `firstUpdated` does important setup:
- Resolves the parent `<form>` reference
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
import { PktSlotController } from '@/controllers/pkt-slot-controller'
import '@/components/input-wrapper'

@customElement('pkt-textinput')
export class PktTextinput extends PktInputElement<Props> {
  inputRef: Ref<HTMLInputElement> = createRef()
  private helptextSlot: Ref<HTMLElement> = createRef()

  @property({ type: String, reflect: true }) value: string = ''

  constructor() {
    super()
    this.slotController = new PktSlotController(this, this.helptextSlot)
  }

  attributeChangedCallback(name: string, _old: string, value: string): void {
    if (name === 'value' && this.value !== _old) {
      this.valueChanged(value, _old)
    }
    super.attributeChangedCallback(name, _old, value)
  }

  render() {
    return html`
      <pkt-input-wrapper
        ?disabled=${this.disabled}
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
          ?disabled=${this.disabled}
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
        <div class="pkt-contents" ${ref(this.helptextSlot)} name="helptext" slot="helptext"></div>
      </pkt-input-wrapper>
    `
  }
}
```

Key patterns:
- **`inputRef`** — Ref to the native `<input>` for validation
- **`e.stopImmediatePropagation()`** — prevents duplicate events bubbling from both native input and custom element
- **`pkt-input-wrapper`** — wraps the input with label, helptext, error display
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
@customElement('pkt-select')
export class PktSelect extends PktOptionsInputElement<{}, TSelectOption> {
  inputRef: Ref<HTMLSelectElement> = createRef()
  private helptextSlot: Ref<HTMLElement> = createRef()

  @property({ type: String }) value: string = ''

  constructor() {
    super()
    this.optionsController = new PktOptionsSlotController(this)
    this.slotController = new PktSlotController(this, this.helptextSlot)
    this.slotController.skipOptions = true  // Don't treat <option> as slot content
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
```

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
