# Tag

Tag displays a short label, status, or category. It can include an icon and an optional close button. Use tags for filtering, categorization, or status indicators.

## Availability

| Package        | Available | Tag / Import                                                                                |
| -------------- | --------- | ------------------------------------------------------------------------------------------- |
| React          | Yes       | `<PktTag>` — `import { PktTag } from '@oslokommune/punkt-react'`                            |
| Elements       | Yes       | `<pkt-tag>` — `import '@oslokommune/punkt-elements/dist/pkt-tag.js'`                        |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/19/elements/pkt-tag.js" type="module">` |

Dark mode: Only the default `gray` skin. Every other skin keeps its light appearance in dark mode.

## Variants

### Skins

Each colour comes in a strong variant and a `-light` variant. Blue additionally has `blue-pale`.

| Skin             | Use                                        |
| ---------------- | ------------------------------------------ |
| `gray` (default) | Standard tag, the only skin with dark mode |
| `gray-light`     | Quieter neutral                            |
| `blue-dark`      | Strong emphasis, light text on dark fill   |
| `blue`           | Standard blue                              |
| `blue-light`     | Lighter blue                               |
| `blue-pale`      | Faintest blue                              |
| `green`          | Positive status or category                |
| `green-light`    | Lighter positive                           |
| `red`            | Negative status or warning                 |
| `red-light`      | Lighter negative                           |
| `yellow`         | Attention or pending                       |
| `yellow-light`   | Lighter attention                          |
| `beige`          | Neutral alternative                        |
| `beige-light`    | Lighter neutral                            |

All skins except `blue-dark` share the same hover, focus and active appearance.

### Forced states

`state` renders a tag in a given state without the user interacting with it. Intended for
documentation, design reviews and visual regression tests, not for production UI.

| State    | Description         |
| -------- | ------------------- |
| `hover`  | As if hovered       |
| `focus`  | As if focused       |
| `active` | As if being pressed |

### Sizes

| Size               | Use            |
| ------------------ | -------------- |
| `small`            | Compact spaces |
| `medium` (default) | Standard use   |
| `large`            | Emphasis       |

### Text styles

| Style                   | Description    |
| ----------------------- | -------------- |
| `normal-text` (default) | Normal weight  |
| `thin-text`             | Lighter weight |

## Props / Attributes

| Prop (React) | Attribute (Elements) | Type                                                                                                       | Default         | Description                |
| ------------ | -------------------- | ---------------------------------------------------------------------------------------------------------- | --------------- | -------------------------- |
| `skin`       | `skin`               | See the Skins table above                                                                                  | `"gray"`        | Color/style of the tag     |
| `state`      | `state`              | `"hover"` \| `"focus"` \| `"active"`                                                                       | —               | Forces a visual state      |
| `size`       | `size`               | `"small"` \| `"medium"` \| `"large"`                                                                       | `"medium"`      | Tag size                   |
| `iconName`   | `iconName`           | string (icon name)                                                                                         | —               | Icon displayed in the tag  |
| `closeTag`   | `closeTag`           | boolean                                                                                                    | `false`         | Show a close/remove button |
| `textStyle`  | `textStyle`          | `"thin-text"` \| `"normal-text"`                                                                           | `"normal-text"` | Text weight style          |
| `type`       | `type`               | `"button"` \| `"submit"` \| `"reset"`                                                                      | `"button"`      | Button type when closable  |
| `ariaLabel`  | `ariaLabel`          | string                                                                                                     | —               | Screen reader label        |

## Slots

| Slot    | Description |
| ------- | ----------- |
| default | Tag text    |

## Examples

### React

```jsx
import { PktTag } from '@oslokommune/punkt-react'

{
  /* Basic tag, default gray skin */
}
;<PktTag>Active</PktTag>

{
  /* With icon */
}
;<PktTag skin="green" iconName="check">
  Completed
</PktTag>

{
  /* Closable tag */
}
;<PktTag skin="beige" closeTag onClick={() => removeTag(id)}>
  Filter item
</PktTag>

{
  /* Small tag */
}
;<PktTag size="small" skin="gray">
  Draft
</PktTag>
```

### Elements

```html
<pkt-tag><span>Active</span></pkt-tag>

<pkt-tag skin="green" iconName="check"><span>Completed</span></pkt-tag>

<pkt-tag skin="beige" closeTag><span>Filter item</span></pkt-tag>

<pkt-tag skin="red-light" state="hover"><span>Hover, for documentation</span></pkt-tag>
```
