# Tabs

Tabs organize content into separate views where only one tab panel is visible at a time. Use them for switching between related sections without navigating to a new page. Tabs can also be used as a navigation menu with links.

## Availability

| Package        | Available | Tag / Import                                                                                    |
| -------------- | --------- | ----------------------------------------------------------------------------------------------- |
| React          | Yes       | `<PktTabs>` + `<PktTabItem>` — `import { PktTabs, PktTabItem } from '@oslokommune/punkt-react'` |
| Elements       | Yes       | `<pkt-tabs>` + `<pkt-tab-item>` — `import '@oslokommune/punkt-elements/dist/pkt-tabs.js'`       |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/20/elements/pkt-tabs.js" type="module">`    |

Dark mode: No

## Usage

There are two ways to define tabs:

1. **Slot-based (recommended):** Use `<PktTabItem>` children inside `<PktTabs>`
2. **Props-based (legacy):** Pass a `tabs` array prop to `<PktTabs>`

The slot-based approach is recommended for new code.

## Props / Attributes — PktTabs

| Prop (React)      | Attribute (Elements) | Type               | Default | Description                                                                             |
| ----------------- | -------------------- | ------------------ | ------- | --------------------------------------------------------------------------------------- |
| `arrowNav`        | `arrow-nav`          | boolean            | `true`  | Enable arrow key navigation between tabs. Also enables `role="tab"` and `aria-selected`. Elements accepts `arrow-nav="false"` |
| `disableArrowNav` | `disable-arrow-nav`  | boolean            | `false` | Disable arrow navigation (e.g. for pure navigation menus). Overrides `arrowNav`. Elements accepts `"false"` |
| `separatorIconName` | `separator-icon-name` | string (icon name) | `""`   | Icon rendered as a separator between tabs                                               |
| `separatorIconPath` | `separator-icon-path` | string           | `""`    | Image path for the separator. Overrides `separatorIconName` when both are set           |
| `togglePanels`    | `toggle-panels`      | boolean            | `false` | Tabs track the active tab themselves and show the panel each tab points at with `controls`. See [Panel switching](#panel-switching). Elements accepts `"false"` |
| `ariaLabelledBy`  | `aria-labelledby`    | string             | —       | ID of a visible heading that names the tabs. Set on the tab list                       |
| `ariaLabel`       | `aria-label`         | string             | —       | Name for the tabs when there is no visible heading. Set on the tab list                 |
| `tabs`            | —                    | `IPktTab[]`        | —       | React only, legacy: array of tab definitions (prefer PktTabItem children). Elements only takes `pkt-tab-item` children |

### IPktTab shape (legacy `tabs` prop, React only)

```ts
{
  text: string                         // Tab title (required)
  href?: string                        // URL (renders as link)
  action?: (index: number) => void     // Called with the tab's index when it is selected
  disabled?: boolean                   // Not selectable, skipped by arrow keys
  controls?: string                    // aria-controls ID
  icon?: string                        // Icon name
  active?: boolean                     // Is this the active tab
  tag?: { text: string, skin?: TTagSkin }  // Badge tag, skin defaults to "blue"
}
```

## Props / Attributes — PktTabItem

| Prop (React) | Attribute (Elements) | Type                                                                                      | Default  | Description                                     |
| ------------ | -------------------- | ----------------------------------------------------------------------------------------- | -------- | ----------------------------------------------- |
| `active`     | `active`             | boolean                                                                                   | `false`  | Is this the active tab                          |
| `href`       | `href`               | string                                                                                    | —        | URL — renders the tab as a link                 |
| `icon`       | `icon`               | string (icon name)                                                                        | —        | Icon displayed in the tab                       |
| `tag`        | `tag`                | string                                                                                    | —        | Badge tag text                                  |
| `tagSkin`    | `tag-skin`           | `TTagSkin` (every Tag skin)                                                               | `"blue"` | Badge tag color                                 |
| `index`      | `index`              | number                                                                                    | `0`      | Tab index, set it on every tab. Arrow keys skip indexes that no tab has |
| `controls`   | `controls`           | string                                                                                    | —        | `aria-controls` ID for the tab panel. With `togglePanels`, that element is shown when the tab is active and hidden otherwise |
| `disabled`   | `disabled`           | boolean                                                                                   | `false`  | Disables the tab — not clickable, skipped by arrow navigation |

## Events

| Event (React)          | Event (Elements) | Description                                    |
| ---------------------- | ---------------- | ---------------------------------------------- |
| `onTabSelected`        | `tab-selected`   | Fires when a tab is selected by click or keyboard, not for a disabled tab. React: called with the index. Elements: the index is in `event.detail.index` |
| `onClick` (PktTabItem) | `click`          | Click on a tab. React: prop on PktTabItem. Elements: the native click event, which doesn't bubble out of a disabled tab |

Elements exports `IPktTabsAttributes` and `IPktTabItemAttributes` with the kebab-case attribute names, for typing templates.

## Slots

| Component  | Slot    | Description            |
| ---------- | ------- | ---------------------- |
| PktTabs    | default | PktTabItem children    |
| PktTabItem | default | Tab label text/content |

## Keyboard and ARIA

Tabs follow the WAI-ARIA Authoring Practices tabs pattern:

- Tab moves into the tab list on the active tab (roving `tabindex`), and on to the panel. With no active tab, the first enabled tab is the one reachable with Tab
- ArrowLeft/ArrowRight move focus and wrap around; Home/End go to the first/last tab; disabled tabs are skipped
- Enter, Space or ArrowDown select the focused tab
- With `togglePanels`, arrow keys, Home and End also select (automatic activation). Without it they only move focus, so a consumer that loads content on select is not triggered by every key press
- Name the tab list with `ariaLabelledBy` (visible heading) or `ariaLabel`
- With `disableArrowNav` (navigation semantics) the active item gets `aria-current="page"` (links) or `aria-current="true"` (buttons) instead of `aria-selected`
- The focus ring only shows for keyboard focus (`:focus-visible`). A tab selected with the mouse shows only the light blue line

## Panel switching

Without `togglePanels`, the consumer owns everything: `active` is a prop, a click only fires `tab-selected` / `onTabSelected`, and `controls` only sets `aria-controls`. The consumer keeps the active index in state and hides the panels.

With `togglePanels`:

- A click, Enter, Space, ArrowDown, the arrow keys, Home or End make the tab active. In Elements this moves the `active` attribute; in React it is internal state that starts from the `active` props and follows them when they change
- Each tab with `controls` sets `hidden` on that element: removed while the tab is active, set otherwise
- With tab semantics (`arrowNav`, the default) the panel gets `role="tabpanel"`, `aria-labelledby` and `tabindex="0"` where each is missing, and the tab gets the id `<controls>-tab`. These ids only render in this mode
- `tab-selected` / `onTabSelected` still fire. Setting `active` from code still switches the tab
- Put `hidden` on the inactive panels in the markup, so they don't show before JS loads
- Panels are found by id in the tab's root node, so tabs and panels must share a document or shadow root
- The event handler cannot stop the switch. If you need that (unsaved changes, for example), leave `togglePanels` off and control `active` yourself

Use it for in-page panels. For a navigation menu (`disableArrowNav`, `href`) leave it off.

## Examples

### Panel switching — React

```jsx
<PktTabs togglePanels>
  <PktTabItem index={0} controls="panel-overview" active>Overview</PktTabItem>
  <PktTabItem index={1} controls="panel-details">Details</PktTabItem>
</PktTabs>

<div id="panel-overview">Overview content</div>
<div id="panel-details" hidden>Details content</div>
```

### Panel switching — Elements / HTML

```html
<pkt-tabs toggle-panels>
  <pkt-tab-item index="0" controls="panel-overview" active>Overview</pkt-tab-item>
  <pkt-tab-item index="1" controls="panel-details">Details</pkt-tab-item>
</pkt-tabs>

<div id="panel-overview">Overview content</div>
<div id="panel-details" hidden>Details content</div>
```

### React — slot-based (recommended)

```jsx
import { PktTabs, PktTabItem } from '@oslokommune/punkt-react'

const [activeTab, setActiveTab] = useState(0)

<PktTabs onTabSelected={(index) => setActiveTab(index)}>
  <PktTabItem active={activeTab === 0} index={0}>Overview</PktTabItem>
  <PktTabItem active={activeTab === 1} index={1}>Details</PktTabItem>
  <PktTabItem active={activeTab === 2} index={2} tag="3" tagSkin="red">Notifications</PktTabItem>
</PktTabs>

{activeTab === 0 && <div>Overview content</div>}
{activeTab === 1 && <div>Details content</div>}
{activeTab === 2 && <div>Notifications content</div>}
```

### React — navigation tabs

```jsx
<PktTabs disableArrowNav>
  <PktTabItem href="/dashboard" active={pathname === '/dashboard'} index={0}>
    Dashboard
  </PktTabItem>
  <PktTabItem href="/reports" active={pathname === '/reports'} index={1}>
    Reports
  </PktTabItem>
  <PktTabItem href="/settings" active={pathname === '/settings'} index={2}>
    Settings
  </PktTabItem>
</PktTabs>
```

### Elements

```html
<pkt-tabs>
  <pkt-tab-item active index="0">Overview</pkt-tab-item>
  <pkt-tab-item index="1">Details</pkt-tab-item>
  <pkt-tab-item index="2" tag="3" tag-skin="red">Notifications</pkt-tab-item>
</pkt-tabs>
```
