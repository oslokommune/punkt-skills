# Header Menu

Header Menu is the global mega menu for Oslo kommune. It renders the `megamenu` slice of the shared header/footer payload — the same endpoint the global header and footer use — so the navigation stays in sync without the consuming site maintaining it.

**Most solutions do not need this component directly.** `<PktHeader type="global">` already renders the mega menu for you. Reach for Header Menu only when you are building a custom header shell and need the menu on its own.

## Availability

| Package        | Available | Tag / Import                                                                                        |
| -------------- | --------- | ----------------------------------------------------------------------------------------------------- |
| React          | Yes       | `<PktHeaderMenu>` — `import { PktHeaderMenu } from '@oslokommune/punkt-react'`                      |
| Elements       | Yes       | `<pkt-header-menu>` — `import '@oslokommune/punkt-elements/dist/pkt-header-menu.js'`                |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/latest/elements/pkt-header-menu.js" type="module">` |

Dark mode: Yes

## Props / Attributes

| Prop (React)       | Attribute (Elements) | Type                                | Default          | Description                                            |
| ------------------ | -------------------- | ----------------------------------- | ---------------- | ------------------------------------------------------ |
| `open`             | `open`               | boolean                             | `false`          | Whether the menu is visible. You control this          |
| `dataUrl`          | `data-url`           | string                              | Oslo kommune CDN | Endpoint to fetch the payload from                     |
| `data`             | `.data` (property)   | `THeaderFooterApi`                  | —                | Pre-fetched payload. When set, no fetch is made        |
| `locale`           | `locale`             | `"nb-NO"` \| `"en-GB"` \| string    | `"nb-NO"`        | Which locale to select from the payload                |
| `mobileBreakpoint` | `mobile-breakpoint`  | number                              | `768`            | Width below which sections collapse into accordions    |

## Events

| Event (React)  | Event (Elements) | Description                                 |
| -------------- | ---------------- | ------------------------------------------- |
| `onDataLoaded` | `data-loaded`    | Payload was fetched, or `data` was supplied |
| `onDataError`  | `data-error`     | Fetching the payload failed                 |

## You own open state and focus

The component is presentational. It does not trap focus, lock scrolling, or decide when to open — the parent header does all of that. If you use Header Menu standalone you are responsible for:

- toggling `open`
- moving focus into the menu when it opens and restoring it on close
- locking body scroll while it is open
- closing on `Escape` and on outside clicks

This is the main reason to prefer `<PktHeader type="global">`, which handles all of it.

## Content Security Policy

Header Menu fetches from `https://cdn.web.oslo.kommune.no/` — a different domain from the rest of Punkt:

```
connect-src 'self' https://punkt-cdn.oslo.kommune.no/ https://cdn.web.oslo.kommune.no/;
```

If you set your own `dataUrl`, allow that URL instead. Fetching the payload server-side and passing it as `data` avoids the requirement entirely. The global header and footer share the same endpoint and the same cached request.

## Accessibility

- Sections collapse into accordions below `mobileBreakpoint`
- The menu must be reachable and dismissible by keyboard — since focus handling is yours, test `Tab` and `Escape` explicitly
- Announce the open/closed state on the control that toggles it, e.g. with `aria-expanded`

## Examples

### React

```jsx
import { PktHeaderMenu } from '@oslokommune/punkt-react'

const [open, setOpen] = useState(false)

;<>
  <button aria-expanded={open} onClick={() => setOpen(!open)}>
    Menu
  </button>
  <PktHeaderMenu open={open} onDataError={(error) => reportError(error)} />
</>
```

Passing a payload you already fetched server-side:

```jsx
<PktHeaderMenu open={open} data={headerFooterData} locale="en-GB" />
```

### Elements

```html
<pkt-header-menu open></pkt-header-menu>

<script>
  const menu = document.querySelector('pkt-header-menu')
  menu.addEventListener('data-error', (e) => console.warn(e.detail.error))

  // Toggle from your own control
  document.querySelector('#menu-button').addEventListener('click', () => {
    menu.open = !menu.open
  })
</script>
```
