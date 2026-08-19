# Footer

Footer is part of the standard page template and is placed at the bottom of the page. It gives the user access to important links like contact information, privacy, and accessibility. Together with the header, the footer establishes Oslo municipality as the sender.

## Availability

| Package        | Available | Tag / Import                                                                                                  |
| -------------- | --------- | ------------------------------------------------------------------------------------------------------------- |
| React          | Yes       | `<PktFooter>` / `<PktFooterSimple>` — `import { PktFooter, PktFooterSimple } from '@oslokommune/punkt-react'` |
| Elements       | Yes       | `<pkt-footer>` / `<pkt-footer-simple>` — `import '@oslokommune/punkt-elements/dist/pkt-footer.js'`            |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/latest/elements/pkt-footer.js" type="module">`                |

One import registers both elements — there is no separate `pkt-footer-simple.js`.

Dark mode: Yes

## How the footer is composed

The footer renders in two bands:

1. **The global (blue) footer** — rendered automatically. Its content (contact details, privacy and accessibility links, social media) comes from Oslo kommune's shared header/footer payload, the same endpoint `PktHeader type="global"` uses. You do not supply this content.
2. **A secondary (grey) band** — your own content, stacked on top. Optional.

With no custom content you get only the global footer. This is the intended setup for most solutions: render the component with no props and the required links stay correct without you maintaining them.

If the payload cannot be fetched, the global footer is hidden silently and only your custom content renders.

## Variants

| Variant            | Use                                                                     |
| ------------------ | ----------------------------------------------------------------------- |
| `PktFooter`        | Custom content as 1–3 columns of links in the secondary band            |
| `PktFooterSimple`  | Custom content as one compact row of links in the secondary band        |

Use `PktFooterSimple` for standalone solutions with limited navigation needs.

## Usage guidelines

**Use footer when:**

- You're building solutions where Oslo municipality is the sender (in practice, footer should always be used)
- You want to make it clear the solution belongs to Oslo municipality

**Content guidelines:**

- Privacy and accessibility links come from the global footer — do not add your own
- Other links in the footer should also be available elsewhere on the site
- Avoid images or icons in the footer
- Don't fill the footer with too many links

## Props / Attributes

These apply to both `PktFooter` and `PktFooterSimple`.

| Prop (React)        | Attribute (Elements)     | Type                        | Default   | Description                                                                       |
| ------------------- | ------------------------ | --------------------------- | --------- | --------------------------------------------------------------------------------- |
| `data`              | `.data` (property)       | `THeaderFooterApi`          | —         | Pre-fetched payload. When set, no fetch is made                                   |
| `dataUrl`           | `data-url`               | string                      | Oslo kommune CDN | Endpoint to fetch the payload from                                         |
| `locale`            | `locale`                 | `"nb-NO"` \| `"en-GB"` \| string | `"nb-NO"` | Which locale to select from the payload                                     |
| `skipGlobal`        | `skip-global`            | boolean                     | `false`   | Skip the global footer entirely, e.g. when CSP blocks the CDN                     |
| `openLinksInNewTab` | `open-links-in-new-tab`  | boolean                     | `false`   | Open your custom links in a new tab. Does not affect global footer links          |
| `includeConsent`    | `include-consent`        | boolean                     | `false`   | Replace the payload's cookie-settings link with Punkt's own consent dialog. When unset, that link is hidden |
| `hotjarId`          | `hotjar-id`              | string \| null              | `null`    | Passed to the consent dialog                                                      |
| `googleAnalyticsId` | `google-analytics-id`    | string \| null              | `null`    | Passed to the consent dialog                                                      |
| `devMode`           | `dev-mode`               | boolean                     | `false`   | Passed to the consent dialog                                                      |
| `cookieDomain`      | `cookie-domain`          | string \| null              | `null`    | Passed to the consent dialog                                                      |
| `cookieSecure`      | `cookie-secure`          | string \| null              | `null`    | Passed to the consent dialog                                                      |
| `cookieExpiryDays`  | `cookie-expiry-days`     | string \| null              | `null`    | Passed to the consent dialog                                                      |

### PktFooter only

| Prop (React) | Attribute (Elements) | Type                  | Default | Description                                  |
| ------------ | -------------------- | --------------------- | ------- | -------------------------------------------- |
| `columns`    | `.columns`           | `IPktFooterColumn[]`  | —       | 1–3 columns of custom content in the grey band |

### PktFooterSimple only

| Prop (React) | Attribute (Elements) | Type               | Default | Description                                   |
| ------------ | -------------------- | ------------------ | ------- | --------------------------------------------- |
| `links`      | `.links`             | `IPktFooterLink[]` | —       | Compact row of custom links in the grey band  |

### Shapes

```ts
interface IPktFooterColumn {
  title: string
  text?: string             // optional paragraph above the links
  links?: IPktFooterLink[]
}

interface IPktFooterLink {
  href: string
  text: string
  external?: boolean        // appends the top domain in parentheses: "Ledige stillinger (webcruiter.com)"
  openInNewTab?: boolean
}
```

## Events

| Event (React)  | Event (Elements) | Description                                                    |
| -------------- | ---------------- | -------------------------------------------------------------- |
| `onDataError`  | `data-error`     | Fetching the payload failed. The global footer is hidden        |

## Deprecated props (React only)

These still work but log a `console.warn`. They are not available in Elements.

| Deprecated             | Use instead                                                          |
| ---------------------- | -------------------------------------------------------------------- |
| `columnOne`            | `columns` — mapped to the first entry                                |
| `columnTwo`            | `columns` — mapped to the second entry                               |
| `personvernOgInfoLink` | Nothing. The privacy link comes from the global footer               |
| `tilgjengelighetLink`  | Nothing. The accessibility link comes from the global footer         |
| `socialLinks`          | Nothing. The social media row comes from the global footer           |

`personvernOgInfoLink` and `tilgjengelighetLink` are only rendered — in the secondary band — when set to a value other than the historical default.

## Content Security Policy

The footer needs more than the standard Punkt CSP, because the global footer fetches its content from a **different** domain than the rest of Punkt — `cdn.web.oslo.kommune.no`, not `punkt-cdn.oslo.kommune.no`:

```
connect-src 'self' https://punkt-cdn.oslo.kommune.no/ https://cdn.web.oslo.kommune.no/;
```

If you set your own `dataUrl`, allow that URL instead. `PktHeader type="global"` uses the same endpoint, so this one permission covers both.

With `includeConsent`, the consent dialog also loads a script and a stylesheet from that CDN:

```
script-src 'self' https://cdn.web.oslo.kommune.no/;
style-src 'self' https://cdn.web.oslo.kommune.no/;
```

Two ways out if you cannot allow the endpoint: fetch the payload server-side and pass it as `data`, so the browser never contacts the endpoint, or drop the global footer with `skipGlobal`.

## Accessibility

- The footer must be responsive — on mobile and tablet, content is displayed in a single column
- All links must have sufficient spacing and be keyboard-accessible
- Test the footer at different screen sizes, including portrait and landscape orientations

## Examples

### React

```jsx
import { PktFooter, PktFooterSimple } from '@oslokommune/punkt-react'

{
  /* Global footer only — the usual case */
}
;<PktFooter />

{
  /* Global footer with two custom columns above it */
}
;<PktFooter
  columns={[
    {
      title: 'About the service',
      text: 'Apply for a kindergarten place in Oslo.',
      links: [{ href: '/about', text: 'About this service' }],
    },
    {
      title: 'Get help',
      links: [
        { href: '/help', text: 'Help and guides' },
        { href: 'https://webcruiter.com', text: 'Vacancies', external: true },
      ],
    },
  ]}
  includeConsent
  googleAnalyticsId="G-XXXXXXXXXX"
/>

{
  /* Simple footer, global skipped because CSP blocks the CDN */
}
;<PktFooterSimple
  skipGlobal
  links={[
    { href: '/personvern', text: 'Privacy' },
    { href: '/tilgjengelighet', text: 'Accessibility' },
  ]}
/>
```

### Elements

```html
<pkt-footer include-consent google-analytics-id="G-XXXXXXXXXX"></pkt-footer>

<script>
  const footer = document.querySelector('pkt-footer')
  footer.columns = [
    {
      title: 'Get help',
      links: [{ href: '/help', text: 'Help and guides' }],
    },
  ]
  footer.addEventListener('data-error', (e) => console.warn(e.detail.error))
</script>
```

## Notes

For Elements, `columns` and `links` are arrays and must be set as JavaScript properties (or passed as JSON strings in the attribute, which Lit will `JSON.parse`).

A global header and a global footer on the same page share one cached request for the payload, so using both costs a single fetch.
