# Link Card

Link Card is a simple card-style link with a title, optional description, icon, and background color. The entire card is clickable and navigates to the specified URL. Use it for navigation-focused cards where the whole surface is one link.

## Availability

| Package        | Available | Tag / Import                                                                                 |
| -------------- | --------- | -------------------------------------------------------------------------------------------- |
| React          | Yes       | `<PktLinkCard>` — `import { PktLinkCard } from '@oslokommune/punkt-react'`                   |
| Elements       | Yes       | `<pkt-linkcard>` — `import '@oslokommune/punkt-elements/dist/pkt-linkcard.js'`               |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/19/elements/pkt-linkcard.js" type="module">` |

Dark mode: Yes

## Variants

### Skins

Same skins as Card. Without `skin` the link card is transparent, with no padding.

| Skin          | Description                         | Dark mode       |
| ------------- | ----------------------------------- | --------------- |
| `transparent` | No background, no padding (default) | Yes             |
| `gray`        | Gray background                     | Yes             |
| `outlined`    | 4px light gray border               | Yes             |
| `white`       | White background                    | No, stays light |
| `blue`        | Blue background                     | No, stays light |
| `green`       | Green background                    | No, stays light |
| `beige`       | Beige background                    | No, stays light |

Skins without dark mode render `data-mode="light"` on the link, so the card and its icons keep their light-mode colors inside a `data-mode="dark"` area. The background does not change on hover; the text turns blue and the title is underlined.

The title is 24px and the padding 24px at every width (transparent has no padding). The link card is at least 14rem wide, but never wider than a parent that is narrower than that (`min-width: min(14rem, 100%)`). In a parent that shrinks to fit its content it is as wide as its text.

`normal` and `gray-outline` are deprecated names for `transparent` and `outlined`, and still work. `beige-outline` and `no-padding` were removed in Punkt 20: the default (transparent) now has no padding, which is what `no-padding` was meant to do. The `external` prop and its icon were also removed: write in the text where the link leads instead, as the docs recommend.

## Usage guidelines

**Use link card when:**

- You need a navigation card where the entire surface is a single link
- You want a card without complex content — just title, description, and an icon

**Avoid link card when:**

- You need more flexible content (images, tags, actions) — use Card instead
- The card should contain multiple interactive elements

**Icon:** use `arrow-link` when the icon should show that the card leads somewhere, as in a list of links, not `chevron-right`. Otherwise pick an icon that represents the topic.

## Props / Attributes

| Prop (React)   | Attribute (Elements) | Type               | Default | Description                |
| -------------- | -------------------- | ------------------ | ------- | -------------------------- |
| `title`        | `title`              | string             | —       | Card title                 |
| `href`         | `href`               | string             | —       | URL the card links to      |
| `skin`         | `skin`               | See skins table    | —       | Visual style               |
| `iconName`     | `iconName`           | string (icon name) | —       | Icon displayed on the card |
| `openInNewTab` | `openInNewTab`       | boolean            | `false` | Open link in a new tab     |

## Slots

| Slot    | Description                   |
| ------- | ----------------------------- |
| default | Description text for the card |

## Examples

### React

```jsx
import { PktLinkCard } from '@oslokommune/punkt-react'

<PktLinkCard
  title="Getting started"
  href="/docs/getting-started"
  skin="blue"
  iconName="arrow-link"
>
  Learn how to set up Punkt in your project.
</PktLinkCard>

<PktLinkCard
  title="GitHub"
  href="https://github.com/oslokommune/punkt"
  skin="beige"
  iconName="arrow-link"
  openInNewTab
>
  View the source code on GitHub.
</PktLinkCard>
```

### Elements

```html
<pkt-linkcard
  title="Getting started"
  href="/docs/getting-started"
  skin="blue"
  iconName="arrow-link"
>
  <p>Learn how to set up Punkt in your project.</p>
</pkt-linkcard>
```
