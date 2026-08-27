---
name: punkt-components
description: Using Punkt design system components (React and Elements/Web Components). Covers component APIs, props, events, slots, and usage patterns for all Punkt UI components. Use when building UIs with Punkt components.
---

# Punkt Components

Skill for using Punkt design system components. Covers both React (`@oslokommune/punkt-react`) and Elements/Web Components (`@oslokommune/punkt-elements`).

For CSS classes, colors, typography, spacing, grid, and layout, see the **punkt-css** skill.

## Getting started

All usage patterns require Punkt CSS. See the punkt-css skill for setup instructions.

### React (NPM)

```sh
npm add @oslokommune/punkt-react
```

```jsx
import { PktButton } from '@oslokommune/punkt-react';

<PktButton skin="primary" variant="icon-left" iconName="user">
  Click me
</PktButton>
```

Punkt components load icons, SVGs, and other resources from the CDN. If your project uses a Content Security Policy (CSP), you must configure it to allow resources from `https://punkt-cdn.oslo.kommune.no/`. See the [CSP section](#content-security-policy-csp) below.

Punkt React components are pure React — they do not wrap web components, and the package has no runtime dependency on `@oslokommune/punkt-elements`. You can pick React or Elements independently; nothing forces you to ship both.

### Elements (NPM)

```sh
npm add @oslokommune/punkt-elements
```

```js
import '@oslokommune/punkt-elements/dist/pkt-button.js';
```

```html
<pkt-button skin="primary">
  <span>Click me</span>
</pkt-button>
```

For reactive slot content (content that changes programmatically), wrap it in a container element like `<span>` or `<div>`.

### Elements (CDN)

No build step required. Include CSS and component scripts directly:

```html
<link href="https://punkt-cdn.oslo.kommune.no/latest/css/pkt.min.css" rel="stylesheet" />
<script src="https://punkt-cdn.oslo.kommune.no/latest/elements/pkt-button.js" type="module"></script>
```

If using CSP, see the [CSP section](#content-security-policy-csp) below.

## Form field sizes

Form fields accept an `inputSize` prop (`input-size` attribute in Elements) with three values:

| Value      | Use                                                                    |
| ---------- | ---------------------------------------------------------------------- |
| `"medium"` | Default. Standard forms — the right choice unless you have a reason     |
| `"small"`  | Dense interfaces such as toolbars, filter bars and table rows           |
| `"xsmall"` | Very tight spaces where a small field is still too tall                 |

Supported by: combobox, datepicker, searchinput, select, textarea, textinput and timepicker.

```jsx
<PktTextinput label="Search" id="q" inputSize="small" />
```

```html
<pkt-textinput label="Search" id="q" input-size="small"></pkt-textinput>
```

Keep the size consistent across fields that sit next to each other — mixing sizes in one form makes the fields look misaligned. Smaller fields also mean smaller touch targets, so prefer `"medium"` on touch-first interfaces.

## Content Security Policy (CSP)

Punkt components load fonts, icons (SVG), and other resources from `https://punkt-cdn.oslo.kommune.no/`. If the application uses a Content Security Policy, the CSP must allow this origin. This applies to all setup methods (NPM and CDN).

**Required CSP directives:**

```
Content-Security-Policy:
  default-src 'self';
  font-src 'self' https://punkt-cdn.oslo.kommune.no/;
  img-src 'self' https://punkt-cdn.oslo.kommune.no/;
  script-src 'self' https://punkt-cdn.oslo.kommune.no/;
  style-src 'self' 'unsafe-inline' https://punkt-cdn.oslo.kommune.no/;
  connect-src 'self' https://punkt-cdn.oslo.kommune.no/;
```

**Why `'unsafe-inline'` in `style-src`?** Because of the intricacies of SVG styling and certain web component features, Punkt requires `'unsafe-inline'` in the `style-src` directive. Without it, some styles will be blocked by the browser. CSP nonces can be used as an alternative but require server-side configuration.

> **Important:** Always instruct users to configure CSP when setting up Punkt components. Missing CSP configuration is a common cause of broken fonts, missing icons, and invisible components.

### Components needing a second domain

`PktHeader type="global"`, `PktFooter` / `PktFooterSimple`, `PktHeaderMenu`, and `PktConsent` also reach `https://cdn.web.oslo.kommune.no/` — a **different** domain from the one above. The directives above are not enough for them:

| Component                          | Extra directive                                            | Why                                      |
| ---------------------------------- | ---------------------------------------------------------- | ---------------------------------------- |
| Global header, footer, header menu | `connect-src https://cdn.web.oslo.kommune.no/`             | Fetches the shared header/footer payload |
| Consent                            | `script-src` + `style-src` for the same domain             | Loads a script and stylesheet from it    |
| Global header search field         | `form-action https://www.oslo.kommune.no/`                 | Submits a `GET` form to the search target |

They share the endpoint, so one `connect-src` entry covers all three. See [header.md](header.md), [footer.md](footer.md), [header-menu.md](header-menu.md) and [consent.md](consent.md) for details and for the server-side `data` workaround.

## Strings

All UI text comes from a single shared catalogue and can be overridden globally, per
subtree, or per component. See [Strings and overrides](strings.md).

## Components

1. [Accordion](accordion.md)
2. [Alert](alert.md)
3. [Backlink](backlink.md)
4. [Breadcrumbs](breadcrumbs.md)
5. [Button](button.md)
6. [Card](card.md)
7. [Checkbox](checkbox.md)
8. [Combobox](combobox.md)
9. [Consent](consent.md)
10. [Datepicker](datepicker.md)
11. [File Upload](fileupload.md)
12. [Footer](footer.md)
13. [Header Menu](header-menu.md)
14. [Header](header.md)
15. [Icon](icon.md)
16. [Input Wrapper](input-wrapper.md)
17. [Link](link.md)
18. [LinkCard](linkcard.md)
19. [Loader](loader.md)
20. [Messagebox](messagebox.md)
21. [Modal](modal.md)
22. [Progressbar](progressbar.md)
23. [Radio Button](radiobutton.md)
24. [Search Input](searchinput.md)
25. [Select](select.md)
26. [Steps](steps.md)
27. [Switch](switch.md)
28. [Table](table.md)
29. [Tabs](tabs.md)
30. [Tag](tag.md)
31. [Text Input](textinput.md)
32. [Textarea](textarea.md)
33. [Timepicker](timepicker.md)
