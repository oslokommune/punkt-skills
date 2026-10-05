# Compound Components

For parent-child component relationships, use React Context:

```tsx
// Parent creates and provides context
const AccordionContext = createContext<{ name?: string }>({})
export const useAccordionContext = () => useContext(AccordionContext)

export const PktAccordion = ({ name, children, ref, ...props }: IPktAccordion) => (
  <AccordionContext.Provider value={{ name }}>
    <div ref={ref} {...props}>{children}</div>
  </AccordionContext.Provider>
)

// Child consumes context
export const PktAccordionItem = ({ name, ref, ...props }: IPktAccordionItem) => {
  const { name: contextName } = useAccordionContext()
  const actualName = name || contextName  // Props override context
  // ...
}
```

## Option groups (`PktRadioGroup`, `PktCheckboxGroup`)

The groups render `PktInputWrapper` with `hasFieldset` and provide a context (`RadioGroupContext`, `CheckboxGroupContext`) that defaults to `null`. Without a provider `PktRadioButton` and `PktCheckbox` render exactly as before, so their markup does not change.

- Props on the option win: `name ?? group.name`, `defaultChecked ?? group.defaultValue…`
- `hasTile`, `hasError` and `disabled` are OR-ed with the group, for parity with Elements, which cannot tell "not set" from `false`
- A controlled group (`value` set) overrides `checked` on the options and warns in development
- **The group is the form field.** `useOptionGroupField` (`src/hooks/useOptionGroupField.ts`) gives the root element behind `ref` `name`, a readable and writable `value` (computed from the DOM: `input:checked` that is not `:disabled`, in DOM order, like `FormData`) and a `focus()` that focuses the checked or first option. That is what makes `register` in React Hook Form work on the group
- The option always calls its own `onChange`/`onFocus`/`onBlur`, then the group's. The group calls its `onChange` with an event whose `target` and `currentTarget` are the root element, so `event.target.value` is the group value (what `Controller`'s `field.onChange` and Formik read). `onFocus`/`onBlur` only fire when focus enters or leaves the group
- `src/components/radiogroup/RadioGroup.forms.test.tsx` and `CheckboxGroup.forms.test.tsx` run React Hook Form in every pattern. Keep them green
- `PktCheckboxGroup` with `required` sets `setCustomValidity()` on every checkbox while none is checked, after every render, on change and after a form reset
- `src/utils/option-contexts.tsx` runs the same option test alone and in a group
