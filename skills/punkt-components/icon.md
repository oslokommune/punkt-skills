# Icon

Icon displays an SVG icon from the Punkt icon library. Icons are loaded from the Punkt CDN by default and can be sized using CSS classes.

## Availability

| Package        | Available | Tag / Import                                                                                 |
| -------------- | --------- | -------------------------------------------------------------------------------------------- |
| React          | Yes       | `<PktIcon>` — `import { PktIcon } from '@oslokommune/punkt-react'`                           |
| Elements       | Yes       | `<pkt-icon>` — `import '@oslokommune/punkt-elements/dist/pkt-icon.js'`                       |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/latest/elements/pkt-icon.js" type="module">` |

Dark mode: No

## Sizes

| CSS class          | Size   |
| ------------------ | ------ |
| `pkt-icon--small`  | Small  |
| `pkt-icon--medium` | Medium |
| `pkt-icon--large`  | Large  |

Set the size by passing the CSS class via `className` (React) or `class` (Elements).

## Props / Attributes

| Prop (React) | Attribute (Elements) | Type                                                               | Default                                             | Description                   |
| ------------ | -------------------- | ------------------------------------------------------------------ | --------------------------------------------------- | ----------------------------- |
| `name`       | `name`               | string (icon name)                                                 | —                                                   | Name of the icon to display   |
| `path`       | `path`               | string                                                             | `"https://punkt-cdn.oslo.kommune.no/latest/icons/"` | Override the icon source path |
| `className`  | `class`              | `"pkt-icon--small"` \| `"pkt-icon--medium"` \| `"pkt-icon--large"` | —                                                   | CSS class for sizing the icon |

## Examples

### React

```jsx
import { PktIcon } from '@oslokommune/punkt-react'

<PktIcon name="chevron-right" />
<PktIcon name="download" className="pkt-icon--large" />
<PktIcon name="user" className="pkt-icon--small" />
```

### Elements

```html
<pkt-icon name="chevron-right"></pkt-icon>
<pkt-icon name="download" class="pkt-icon--large"></pkt-icon>
<pkt-icon name="user" class="pkt-icon--small"></pkt-icon>
```

## Notes

Icons are fetched from `https://punkt-cdn.oslo.kommune.no/latest/icons/` by default. If you need to serve icons from a different location (e.g. for CSP or offline use), override the `path` prop.
