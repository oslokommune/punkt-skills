# Breadcrumbs

Breadcrumbs show the user where they are in the structure and make it possible to navigate back to a higher level. It helps the user understand the hierarchy and find their way back.

## Availability

| Package        | Available | Tag / Import                                                                     |
| -------------- | --------- | -------------------------------------------------------------------------------- |
| React          | Yes       | `<PktBreadcrumbs>` — `import { PktBreadcrumbs } from '@oslokommune/punkt-react'`                          |
| Elements       | Yes       | `<pkt-breadcrumbs>` — `import '@oslokommune/punkt-elements/dist/pkt-breadcrumbs.js'`                      |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/latest/elements/pkt-breadcrumbs.js" type="module">`       |

Dark mode: Yes

## Usage guidelines

**Use breadcrumbs when:**

- The solution has a clear hierarchy
- You want to show the user where they are in the structure
- The user should be able to jump back to a higher level

**Avoid breadcrumbs when:**

- The hierarchy is shallow — use Backlink instead
- The user's path back is not clear
- Other navigation already covers the need

**Backlink vs Breadcrumbs:** Punkt supports both backlink and breadcrumbs. Be consistent — use only one. If your solution is used with Oslo municipality's public websites (www.oslo.kommune.no), always use backlink, never breadcrumbs.

**Content guidelines:**

- Each level should have a short, descriptive name
- Avoid technical or internal names (e.g. "/nav/root/sub/area")
- The component should be used with a maximum of 4 links

## Props / Attributes

| Prop (React)     | Attribute (Elements) | Type                                                                                                         | Default        | Description                                                                                                          |
| ---------------- | -------------------- | ------------------------------------------------------------------------------------------------------------ | -------------- | -------------------------------------------------------------------------------------------------------------------- |
| `breadcrumbs`    | `breadcrumbs`        | `Array<{ text: string, href: string }>`                                                                      | **(required)** | List of breadcrumb items                                                                                             |
| `navigationType` | —                    | `"router"` \| `"anchor"`                                                                                     | `"anchor"`     | React only. Navigation method — `"anchor"` for standard links, `"router"` for client-side routing                    |
| `renderLink`     | —                    | `(args: { href: string, className: string, children: ReactNode, props: AnchorHTMLAttributes }) => ReactNode` | —              | React only. Custom link component (e.g. Next.js Link). When set, `navigationType` is implicitly `"router"`           |

## Events

| Event (React) | Event (Elements) | Description                                                                                              |
| ------------- | ---------------- | -------------------------------------------------------------------------------------------------------- |
| —             | `navigate`       | Elements only. Fires when a breadcrumb link is clicked. Detail: `{ item, index, originalEvent }`. Call `event.preventDefault()` to prevent navigation |

## React vs Elements: custom navigation

React and Elements handle custom navigation (SPA routers) differently:

**React** uses `renderLink` to replace the link component:
```jsx
<PktBreadcrumbs
  breadcrumbs={crumbs}
  renderLink={({ href, className, children }) => (
    <Link to={href} className={className}>{children}</Link>
  )}
/>
```

**Elements** uses the `navigate` event. Call `preventDefault()` to stop the browser from navigating, then handle routing yourself:
```html
<pkt-breadcrumbs id="crumbs"></pkt-breadcrumbs>
<script>
  const el = document.querySelector('#crumbs')
  el.breadcrumbs = [
    { text: 'Hjem', href: '/' },
    { text: 'Tjenester', href: '/tjenester' },
    { text: 'Denne siden', href: '/tjenester/denne' },
  ]
  el.addEventListener('navigate', (e) => {
    e.preventDefault()
    myRouter.push(e.detail.item.href)
  })
</script>
```

If you don't listen to the `navigate` event, links work as normal `<a>` elements with full page navigation.

## Accessibility

- Link text must be clear and describe where the link goes
- Keyboard: navigate with Tab, activate with Enter. The component has a visible focus indicator
- Do not use breadcrumbs as the only way to navigate back — it should supplement the main navigation
- The last item in the trail (current page) is not clickable

## Examples

### React

```jsx
import { PktBreadcrumbs } from '@oslokommune/punkt-react'

{
  /* Basic breadcrumbs */
}
;<PktBreadcrumbs
  breadcrumbs={[
    { text: 'Home', href: '/' },
    { text: 'Services', href: '/services' },
    { text: 'Current page' },
  ]}
/>

{
  /* With React Router */
}
;<PktBreadcrumbs breadcrumbs={crumbs} navigationType="router" />

{
  /* With custom link component (e.g. Next.js) */
}
;<PktBreadcrumbs
  breadcrumbs={crumbs}
  renderLink={({ href, className, children }) => (
    <Link to={href} className={className}>
      {children}
    </Link>
  )}
/>
```

### Elements / HTML

```html
<pkt-breadcrumbs id="crumbs"></pkt-breadcrumbs>
<script>
  document.querySelector('#crumbs').breadcrumbs = [
    { text: 'Hjem', href: '/' },
    { text: 'Tjenester', href: '/tjenester' },
    { text: 'Denne siden', href: '/tjenester/denne' },
  ]
</script>
```

Or set breadcrumbs as a JSON attribute:
```html
<pkt-breadcrumbs breadcrumbs='[{"text":"Hjem","href":"/"},{"text":"Tjenester","href":"/tjenester"},{"text":"Denne siden","href":"/tjenester/denne"}]'></pkt-breadcrumbs>
```
