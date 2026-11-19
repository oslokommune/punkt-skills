# Icon

Icon displays an SVG icon from the Punkt icon library. Icons are loaded from the Punkt CDN by default and can be sized with the `size` prop.

## Availability

| Package        | Available | Tag / Import                                                                                 |
| -------------- | --------- | -------------------------------------------------------------------------------------------- |
| React          | Yes       | `<PktIcon>` — `import { PktIcon } from '@oslokommune/punkt-react'`                           |
| Elements       | Yes       | `<pkt-icon>` — `import '@oslokommune/punkt-elements/dist/pkt-icon.js'`                       |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/20/elements/pkt-icon.js" type="module">` |

Dark mode: No

## Sizes

| `size`    | Size                         | Class               |
| --------- | ---------------------------- | ------------------- |
| `small`   | 16px                         | `pkt-icon--small`   |
| `medium`  | 24px                         | `pkt-icon--medium`  |
| `large`   | 64px                         | `pkt-icon--large`   |
| `inherit` | 1em, follows the text size   | `pkt-icon--inherit` |

Set the size with the `size` prop/attribute; it adds the class `pkt-icon--{size}`. Without `size`, the surrounding styles decide the size. Plain HTML/CSS can use the classes directly.

`inherit` makes the icon as tall as the surrounding text and centres it on the line (`vertical-align: middle`), down to small text sizes; the other sizes never go below 16px. For another size, set the font size on the icon itself, e.g. `size="inherit" class="pkt-txt-24"`. It also works as `.pkt-link__icon` in icon links.

## Props / Attributes

| Prop (React) | Attribute (Elements) | Type                                                               | Default                                             | Description                   |
| ------------ | -------------------- | ------------------------------------------------------------------ | --------------------------------------------------- | ----------------------------- |
| `name`       | `name`               | string (icon name)                                                 | —                                                   | Name of the icon to display   |
| `path`       | `path`               | string                                                             | `"https://cdn.ikoner.oslo.kommune.no/icons/"`       | Override the icon source path. Can also be set globally with `window.pktIconPath` |
| `size`       | `size`               | `"small"` \| `"medium"` \| `"large"` \| `"inherit"`                | —                                                   | Icon size, adds `pkt-icon--{size}` |
| `ariaHidden` | `aria-hidden`        | boolean                                                            | —                                                   | Hides the icon from screen readers. When unset, the icon is hidden unless `ariaLabel` is set |
| `ariaLabel`  | `aria-label`         | string                                                             | —                                                   | Accessible name. When set, the icon is exposed to screen readers with this text |

## Examples

### React

```jsx
import { PktIcon } from '@oslokommune/punkt-react'

<PktIcon name="chevron-right" />
<PktIcon name="download" size="large" />
<PktIcon name="user" size="small" />
<p className="pkt-txt-14-light">Next <PktIcon name="arrow-right" size="inherit" /></p>
```

### Elements

```html
<pkt-icon name="chevron-right"></pkt-icon>
<pkt-icon name="download" size="large"></pkt-icon>
<pkt-icon name="user" size="small"></pkt-icon>
<p class="pkt-txt-14-light">Next <pkt-icon name="arrow-right" size="inherit"></pkt-icon></p>
```

## Notes

Icons are fetched from Oslo kommune's icon library, `https://cdn.ikoner.oslo.kommune.no/icons/`, by default (before Punkt 20 they came from `https://punkt-cdn.oslo.kommune.no/latest/icons/`). A Content Security Policy must allow that origin in `connect-src`, since the icons are fetched with `fetch()`. If you need to serve icons from a different location (e.g. for CSP or offline use), override the `path` prop. The loading animations in Loader and Button (`loader`, `spinner`, `spinner-blue`) still come from `https://punkt-cdn.oslo.kommune.no/latest/animations/` (`window.pktAnimationPath`).

## Available icons

These are all the available icons. Do NOT use any other icons unless you use path override and point to an SVG you already know about.

'24h' | 'access-prohibited' | 'accessibility' | 'accessible-water-access' | 'achievement' | 'active-person' | 'adjust' | 'advice' | 'alcohol-prohibited' | 'alert-circle' | 'alert-error-system' | 'alert-error' | 'alert-information' | 'alert-success-system' | 'alert-success' | 'alert-warning' | 'ambulance' | 'apple' | 'archive' | 'arrow-circle' | 'arrow-directions' | 'arrow-down-left' | 'arrow-down-right' | 'arrow-down' | 'arrow-left' | 'arrow-link' | 'arrow-return' | 'arrow-right' | 'arrow-up-left' | 'arrow-up-right' | 'arrow-up' | 'arrow' | 'arts-and-culture' | 'attachment' | 'audioguide' | 'baby-bottle' | 'baby-changing-room' | 'backpack-prohibited' | 'backpack' | 'ball-sports' | 'balloons' | 'bank-card' | 'bath' | 'bed' | 'beehive' | 'bicycle-parking' | 'bicycle-prohibited' | 'bicycle-service' | 'bicycle' | 'books' | 'boys-shower' | 'boys-toilet' | 'brain-heart' | 'brain-signal' | 'brain' | 'briefcase' | 'bulb' | 'bullseye' | 'bus' | 'cafe' | 'calendar' | 'camera-surveillance-CCTV' | 'camera' | 'campfire-site' | 'camping-prohibited' | 'camping' | 'car' | 'celebration' | 'character-exclamation-mark' | 'character-information' | 'charging-pile' | 'charging-point' | 'check-big' | 'check-box-checked' | 'check-box-unchecked' | 'check-circle' | 'check-medium' | 'check' | 'cherry-blossom' | 'chevron-down' | 'chevron-left' | 'chevron-right' | 'chevron-thin-down' | 'chevron-thin-left' | 'chevron-thin-right' | 'chevron-thin-up' | 'chevron-up-and-down' | 'chevron-up' | 'childrens-toilet' | 'circular-economy' | 'climbing' | 'clipboard' | 'clock-history' | 'clock' | 'close' | 'cloud' | 'code' | 'coffee' | 'cogwheel' | 'coin-stacks' | 'computer' | 'cookie' | 'copy' | 'crane' | 'crayfish' | 'cross-circle-big' | 'cross-circle-fill-large' | 'cross-circle-fill' | 'cross-circle' | 'cross' | 'cupboard' | 'dancing' | 'data' | 'deer' | 'defibrillators' | 'demonstrator' | 'district' | 'diving-prohibited' | 'dock' | 'document-code' | 'document-download' | 'document-edit' | 'document-pdf' | 'document-plain' | 'document-table' | 'document-text' | 'dog-forbidden' | 'dog-on-a-leash' | 'download' | 'drag' | 'drinking-area-refreshments' | 'drone' | 'drugs-prohibited' | 'dump-truck' | 'ecg-heart' | 'edit' | 'education' | 'elderly' | 'electric-car' | 'electric-van' | 'elevator' | 'email' | 'escalator' | 'exclamation-mark-circle' | 'exclamation-mark-square' | 'exhibition' | 'exit-door' | 'exit' | 'expand' | 'eye' | 'face-almost-happy' | 'face-almost-sad' | 'face-happy' | 'face-neutral' | 'face-sad' | 'facebook' | 'factory-fill' | 'factory' | 'feedback' | 'festival-tent' | 'figma' | 'filming-prohibited' | 'filter' | 'fire-prohibited' | 'fire-safety' | 'fireworks-prohibited' | 'fishing-prohibited' | 'fixed-bench' | 'fixed-grill' | 'flag' | 'flammable-waste' | 'flash-prohibited' | 'folder-pending' | 'folder-restricted' | 'folder' | 'food-and-drinks-prohibited' | 'fountain' | 'game-console' | 'gavel' | 'girls-shower' | 'girls-toilet' | 'github' | 'goal' | 'graffiti-allowed' | 'graffiti-prohibited' | 'graph' | 'grid' | 'group-pending' | 'group' | 'gym' | 'handicap-elevator' | 'handicap' | 'hands-globe' | 'headset' | 'heart-filled' | 'heart-hand' | 'heart-plus' | 'heart' | 'height-restriction' | 'helmet' | 'hiking-trail' | 'holding-hands' | 'home' | 'horizontal-menu' | 'house-heart' | 'ice-skating' | 'incoming-mail' | 'indoor-pool' | 'information' | 'instagram' | 'introduced-species' | 'invoice' | 'key' | 'ladies-toilet' | 'language' | 'law-paragraph' | 'lawn-mower' | 'layers' | 'lecture-workplace' | 'lecture' | 'life-ring' | 'lightning-bolt' | 'link' | 'linkedin' | 'list' | 'living-room' | 'location-pin-filled' | 'location-pin' | 'lock-locked' | 'lock-unlocked' | 'luggage-prohibited' | 'magnifying-glass-big' | 'magnifying-glass-small' | 'map-cursor' | 'map-layers' | 'map' | 'matrix' | 'medicine-cabinet' | 'megaphone' | 'memorial-stone' | 'mens-toilet' | 'menu' | 'message' | 'microphone' | 'minimize' | 'minus-circle-fill' | 'minus-circle' | 'minus-sign' | 'mobile-phone' | 'moon' | 'motorcycle' | 'movie-clapper' | 'museum' | 'music-notes' | 'nature-plant-2' | 'nature-plant' | 'network' | 'new-window-small' | 'new-window' | 'no-walking' | 'nodejs' | 'obstacle' | 'organization' | 'outdoor-seating' | 'outgoing-mail' | 'package' | 'paint-bucket' | 'palette' | 'paper-plane-horizontal' | 'paper-plane' | 'paragraph' | 'park' | 'parking-meter' | 'parking' | 'partner-workout' | 'paste' | 'payment-terminal' | 'person' | 'picnic-table' | 'picture' | 'placeholder-icon' | 'planter-box' | 'playground' | 'plus-circle-fill' | 'plus-circle' | 'plus-sign' | 'pothole' | 'pregnant' | 'price-tag' | 'pride-heart' | 'print' | 'privacy' | 'process-back' | 'process-forward' | 'process-iteration' | 'processing' | 'product-display' | 'prohibited' | 'pull-up-bar' | 'punkt-circle-blue' | 'qr-code' | 'question' | 'radio-checked' | 'radio-unchecked' | 'react' | 'receipt' | 'recycling' | 'restaurant' | 'road-signs' | 'roadblock' | 'ruler' | 'running-shoe' | 'sass' | 'satellite' | 'sauna' | 'save' | 'scooter-prohibited' | 'scooter' | 'shapes' | 'share-outline' | 'share' | 'shield-fire' | 'shield-law-paragraph' | 'shield-star' | 'shoes-prohibited' | 'shopping-bag' | 'shower' | 'signature' | 'skateboarding' | 'slack' | 'sledding' | 'smiley-neutral' | 'smiley-sad' | 'smiley-smile' | 'smoking-prohibited' | 'snow-removal-truck' | 'snow' | 'sort' | 'sound-off' | 'sound-on' | 'sparkles' | 'stairs-down' | 'stairs-up' | 'stairs' | 'star-filled' | 'star' | 'stop' | 'street-light' | 'street-vendor' | 'stroller-prohibited' | 'stroller' | 'studded-tires' | 'sun' | 'svg' | 'swimming-area' | 'swimming-ladder' | 'swingset' | 'table' | 'tap' | 'target' | 'thumbs-down' | 'thumbs-up' | 'tickets' | 'toilet-unisex' | 'toilet' | 'toll-gate' | 'tool' | 'touching-prohibited' | 'traffic-light' | 'trail-gate' | 'trail' | 'tram' | 'trash-can' | 'trash' | 'twitter' | 'two-people-dancing' | 'two-people-talking' | 'umbrella-prohibited' | 'urban-landscape' | 'urinal' | 'user' | 'van' | 'vertical-menu' | 'viewpoint' | 'virus' | 'vue' | 'walking' | 'wardrobe-boys' | 'wardrobe-girls' | 'wardrobe' | 'waste-container' | 'water-faucet' | 'wheelchair-ramp' | 'wheelchair' | 'wifi' | 'wine-bottle' | 'workplace' | 'wrench' | 'x' | 'zoom-minus' | 'zoom-plus'
