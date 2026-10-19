# Hooks (shared utilities)

Available in `packages/react/src/hooks/`:

| Hook                                   | Purpose                                     |
| -------------------------------------- | ------------------------------------------- |
| `useElementWidth(ref, fallbackWidth?)` | ResizeObserver-based element width tracking. On the server it returns `fallbackWidth` or `Infinity`, so breakpoint checks render the desktop layout |
| `useGeneratedId(id?)`                  | Returns `id`, or a `useId`-based id that matches between server and client and is safe in CSS selectors |
| `useScrollLock(locked)`                | iOS-safe body scroll locking                |
| `useOwnerForm(ref, formId?)`           | Resolves the `<form>` a control belongs to  |

## `useOwnerForm`

Composite form components (combobox, timepicker, fileupload) render their own hidden input and
listen for the form's `reset`/`submit` events. They must not use `closest('form')` for that — a
control linked with the `form` attribute sits outside the form it belongs to.

```tsx
const ownerForm = useOwnerForm(wrapperRef, form)

useEffect(() => {
  if (!ownerForm) return
  ownerForm.addEventListener('reset', handleReset)
  return () => ownerForm.removeEventListener('reset', handleReset)
}, [ownerForm])
```

Pass `form` through to the hidden input that carries the value, so the browser submits it with the
right form:

```tsx
<input type="hidden" name={name} form={form} value={value} />
```

Simple components (textinput, textarea, checkbox, radio, select) need nothing extra — they spread
`...props` onto a native control, so `form` reaches it already.
