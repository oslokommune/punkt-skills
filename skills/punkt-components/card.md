# Card

Card groups related content together and can contain text, images, icons, buttons, links, or a combination of these. Cards make it easier to scan, compare, and choose between multiple items.

## Availability

| Package        | Available | Tag / Import                                                                             |
| -------------- | --------- | ---------------------------------------------------------------------------------------- |
| React          | Yes       | `<PktCard>` — `import { PktCard } from '@oslokommune/punkt-react'`                       |
| Elements       | Yes       | `<pkt-card>` — `import '@oslokommune/punkt-elements/dist/pkt-card.js'`                   |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/20/elements/pkt-card.js" type="module">` |

Dark mode: Yes

## Variants

### Layouts

| Layout       | Use                                                                      |
| ------------ | ------------------------------------------------------------------------ |
| `vertical`   | Content stacked vertically — works well with multiple cards side by side |
| `horizontal` | When you want to use more of the width, e.g. in desktop view             |

A horizontal card stacks (image above text) when the card itself is narrower than 36rem. This is a container query on the card, so a narrow card stacks even on a wide screen. Browsers without container queries fall back to the viewport width.

The card fills its parent (`width: 100%`). It is at least 14rem wide, but never wider than a parent that is narrower than that (`min-width: min(14rem, 100%)`), so it doesn't overflow narrow columns. In a parent that shrinks to fit its content (`inline-block`, `fit-content`, a flex item without a width, an `auto` grid column) a vertical card asks for 25rem and a horizontal card for 37rem, enough to lie side by side. With less room it gets narrower, and a horizontal card stacks.

### Skins

| Skin          | Use                                                                                                | Dark mode       |
| ------------- | -------------------------------------------------------------------------------------------------- | --------------- |
| `gray`        | Gray background (default)                                                                          | Yes             |
| `outlined`    | Border without background. The image is inset with padding                                         | Yes             |
| `transparent` | No background and no padding. The text lines up with the image edges, 32px below it (or beside it) | Yes             |
| `white`       | White background, for use on colored surfaces                                                      | No, stays light |
| `blue`        | Blue background, to highlight content                                                              | No, stays light |
| `green`       | Green background, to highlight content                                                             | No, stays light |
| `beige`       | Beige background, to highlight content                                                             | No, stays light |

Skins without dark mode render `data-mode="light"` on the card, so the card and everything inside it keep their light-mode colors inside a `data-mode="dark"` area.

### Image

The image has a fixed height that steps up with the width of the card (a container query, not the screen), and is cropped from the center, so cards of the same width get the same image height whatever the source images are. Vertical: 12.5rem, 14.375rem from a card width of 20rem, 18.125rem from 30rem. Horizontal: 11.5rem when stacked, 17.875rem from 36rem, 18.75rem from 48rem (in a row the image also stretches to the height of the text). Use `imageAspect` to use an aspect ratio instead.

### Clickable card

With `clickCardLink`, the heading becomes a link that covers the whole card, and an `arrow-link` icon follows the last word of the heading, inline, so it wraps with the text. On hover the text turns blue and the heading is underlined. Keyboard focus shows a focus ring around the whole card.

## Usage guidelines

**Use card when:**

- You want to group information
- You need clear choices in a grid or list layout
- You want flexible display with or without image, content, or action

**Avoid card when:**

- You're showing just one thing and don't need visual grouping
- The information is better suited for plain text or a table

**Content guidelines:**

- Keep content clean, clear, and concrete — avoid overloading the card
- Use consistent elements across cards in the same view
- Use short titles and clear actions
- If the card is clickable, make it clear where it leads

**Interactive vs non-interactive cards:**

- A card can either be fully clickable (the whole card is a link) or contain clickable elements inside it (buttons, links)
- Do not mix these — don't put buttons inside a clickable card. One action per area.

## Props / Attributes

| Prop (React)       | Attribute (Elements) | Type                                                                              | Default      | Description                                                         |
| ------------------ | -------------------- | --------------------------------------------------------------------------------- | ------------ | ------------------------------------------------------------------- |
| `heading`          | `heading`            | string                                                                            | —            | Card title                                                          |
| `headingLevel`     | `headingLevel`       | `1` \| `2` \| `3` \| `4` \| `5` \| `6`                                            | `3`          | HTML heading level for the title                                    |
| `subheading`       | `subheading`         | string                                                                            | —            | Subtitle displayed below the heading                                |
| `layout`           | `layout`             | `"vertical"` \| `"horizontal"`                                                    | `"vertical"` | Portrait or landscape layout                                        |
| `skin`             | `skin`               | `"gray"` \| `"outlined"` \| `"transparent"` \| `"white"` \| `"blue"` \| `"green"` \| `"beige"` | `"gray"`     | Visual style and color                                              |
| `metaLead`         | `metaLead`           | string                                                                            | —            | Bold metadata text (e.g. author)                                    |
| `metaTrail`        | `metaTrail`          | string                                                                            | —            | Normal-weight metadata text (e.g. date)                             |
| `tagPosition`      | `tagPosition`        | `"top"` \| `"bottom"`                                                             | `"top"`      | Position of tags in the card                                        |
| `clickCardLink`    | `clickCardLink`      | string                                                                            | —            | URL to make the entire card clickable. Heading is used as link text |
| `openLinkInNewTab` | `openLinkInNewTab`   | boolean                                                                           | `false`      | Open the card link in a new tab                                     |
| `image`            | `image`              | `{ src: string, alt?: string }`                                                   | —            | Card image. Only set `alt` for non-decorative images                |
| `imageAspect`      | `imageAspect`        | `"auto"` \| `"1-1"` \| `"4-3"` \| `"3-2"` \| `"16-9"` \| `"19-10"`                | —            | Use an aspect ratio instead of the fixed image height               |
| `zoom`             | `zoom`               | boolean                                                                           | `false`      | Zoom the image slightly on hover                                    |
| `tags`             | `tags`               | `Array<{ text: string, skin?: string, iconName?: string, ariaLabel?: string }>`   | —            | List of tags displayed on the card                                  |
| `ariaLabel`        | `ariaLabel`          | string                                                                            | —            | Screen reader label. Defaults to heading or subheading              |

## Slots

| Slot    | Description         |
| ------- | ------------------- |
| default | Content of the card |

## Accessibility

- Use the correct `headingLevel` for the context where the card is displayed — all cards at the same level should use the same heading level
- Interactive cards: the whole card is a tab stop, activated with Enter or Space
- Non-interactive cards: only buttons and links inside are focusable
- All focusable elements have a visible focus indicator
- Write clear link text that screen readers understand
- Only set `image.alt` for images that convey meaning — decorative images should omit it

## Examples

### React

```jsx
import { PktCard } from '@oslokommune/punkt-react'

{
  /* Basic card */
}
;<PktCard heading="My service" subheading="A brief description">
  <p>Card content goes here.</p>
</PktCard>

{
  /* Clickable card with image */
}
;<PktCard
  heading="Service name"
  clickCardLink="/services/detail"
  image={{ src: '/images/service.jpg', alt: 'Service illustration' }}
  skin="blue"
  tags={[{ text: 'New', skin: 'green' }]}
/>

{
  /* Horizontal layout */
}
;<PktCard
  heading="Article title"
  layout="horizontal"
  metaLead="Author name"
  metaTrail="2024-01-15"
  image={{ src: '/images/article.jpg' }}
>
  <p>Article summary text.</p>
</PktCard>
```

### Elements

```html
<pkt-card heading="My service" subheading="A brief description">
  <p>Card content goes here.</p>
</pkt-card>

<pkt-card heading="Service name" clickCardLink="/services/detail" skin="blue"> </pkt-card>

<pkt-card heading="Article title" layout="horizontal">
  <p>Article summary text.</p>
</pkt-card>
```

## Notes

The card redesign removed `padding`, `imageShape`, `borderOnHover`, `restrictImageHeight` and the `outlined-beige` skin. When migrating:

- `padding`: remove it. Padding now follows the skin; only `outlined` has space around the image.
- `imageShape="round"`: remove it. Images are always rectangular.
- `borderOnHover`: remove it. Clickable cards show hover with blue text and an underlined heading.
- `restrictImageHeight`: remove it. The fixed image height is now the default.
- `skin="outlined-beige"`: use `outlined`. The old default was `outlined`; omit `skin` to get the new default `gray`.

For Elements, complex props like `image` and `tags` can be passed in several ways: as JavaScript properties, as JSON strings in HTML attributes (Lit will `JSON.parse` them internally), or directly as objects in frameworks like Vue that bind to properties.

```js
// As JavaScript properties
const card = document.querySelector('pkt-card')
card.image = { src: '/images/service.jpg', alt: 'Service illustration' }
card.tags = [{ text: 'New', skin: 'green' }]
```

```html
<!-- As JSON string attribute -->
<pkt-card image='{"src": "/images/service.jpg", "alt": "Service illustration"}'></pkt-card>
```
