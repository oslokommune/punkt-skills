# Getting Started

## Setup Methods

### CDN (recommended for quick start)

Add the stylesheet to your HTML `<head>`. Fonts are served from the CDN automatically.

```html
<link href="https://punkt-cdn.oslo.kommune.no/latest/css/pkt.min.css" rel="stylesheet" />
```

This includes everything: normalise, base styles, element styles, and component styles.

To override Punkt from your own CSS without fighting specificity, use `pkt.layer.min.css` instead — see [Using CSS `@layer`](#using-css-layer).

### SCSS (embedded)

For projects with a Sass build step. This gives access to SCSS variables, mixins, and functions.

**Install:**

```sh
npm add @oslokommune/punkt-css @oslokommune/punkt-assets
npm add -D sass
```

**Import in your main SCSS file:**

```scss
/* Override font path to point to the punkt-assets package */
@use '@oslokommune/punkt-css/dist/scss/abstracts/variables' with (
  $font-path: '@oslokommune/punkt-assets/dist'
);

/* Import all Punkt styles */
@use '@oslokommune/punkt-css/dist/scss/pkt';
```

The `$font-path` override is required so the font-face declarations resolve to the correct package.

## Using CSS `@layer`

Put Punkt in a CSS cascade layer and your own styles win without you having to make your selectors more specific. This is the recommended way to override Punkt.

### CDN

The simplest option is the pre-wrapped stylesheet, which is all of Punkt already inside `@layer punkt`:

```html
<link href="https://punkt-cdn.oslo.kommune.no/latest/css/pkt.layer.min.css" rel="stylesheet" />
```

Then write your own styles in a later layer:

```css
@layer punkt, app;

@layer app {
  .my-component {
    color: red;
  }
}
```

Declaring `@layer punkt, app;` up front fixes the order regardless of which stylesheet the browser parses first. Without it, order depends on which layer the browser sees first, so a stylesheet loaded before the Punkt `<link>` would lose.

You can also import the regular file into a layer yourself, which is what you need if you only want parts of Punkt:

```css
@layer punkt, app;

@import url('https://punkt-cdn.oslo.kommune.no/latest/css/pkt-normalise.min.css') layer(punkt);
@import url('https://punkt-cdn.oslo.kommune.no/latest/css/pkt-base.min.css') layer(punkt);
@import url('https://punkt-cdn.oslo.kommune.no/latest/css/pkt-elements.min.css') layer(punkt);
@import url('https://punkt-cdn.oslo.kommune.no/latest/css/pkt-components.min.css') layer(punkt);
```

Keep that file order — it matches the order inside `pkt.min.css`.

### SCSS

Use `pkt.layer` instead of `pkt`. Everything else stays the same:

```scss
/* Override font path to point to the punkt-assets package */
@use '@oslokommune/punkt-css/dist/scss/abstracts/variables' with (
  $font-path: '@oslokommune/punkt-assets/dist'
);

/* Punkt styles, wrapped in @layer punkt */
@use '@oslokommune/punkt-css/dist/scss/pkt.layer';

@layer app {
  .my-component {
    color: red;
  }
}
```

### Cascade rules to know

- **The last layer wins.** In `@layer punkt, app;`, `app` beats `punkt` no matter how specific Punkt's selectors are.
- **Unlayered styles beat every layer.** Any CSS you write outside a layer wins over all layered styles, including your own `app` layer.
- **Put all of Punkt in one layer.** If you split Punkt across several layers, or leave some files unlayered, Punkt's own styles start competing with each other — the link styles in `pkt-base` could beat the button styles in `pkt-components`.

## What pkt.css includes

The full stylesheet is composed of four parts, loaded in order:

1. **Normalise** — CSS reset (box-sizing, form normalization, reduced motion support)
2. **Base** — Fonts, colors, typography classes, spacing utilities, containers, grid, layouts, visibility, accessibility helpers
3. **Elements** — Styled HTML elements (buttons, inputs, checkboxes, selects, tables, etc.)
4. **Components** — Complex UI components (accordion, alert, header, footer, modal, tabs, etc.)

These are ordering groups inside the stylesheet, not CSS `@layer` layers — see [Using CSS `@layer`](#using-css-layer) for those.

## Modular CSS

If you don't need all of Punkt, these entry points ship as their own stylesheets:

| File | Contents |
|---|---|
| `pkt-tokens.min.css` | Color variables, dark mode, `@font-face` |
| `pkt-utilities.min.css` | Spacing, color and visibility helper classes |
| `pkt-grid.min.css` | Containers, grid and layouts |
| `pkt-normalise.min.css` | Reset only |
| `pkt-base.min.css` | Base styles only |
| `pkt-elements.min.css` | Styled HTML elements only |
| `pkt-components.min.css` | Components only |

`pkt-utilities` and `pkt-grid` both build on the custom properties in `pkt-tokens`, so load `pkt-tokens` first.

Per-component files also exist (`components/alert.min.css`, `elements/button.min.css`), but they are less commonly used — reach for the grouped entries above first.

## Using SCSS abstracts

When using the SCSS method, you can import abstracts (variables, mixins, functions) in your own stylesheets without pulling in any CSS output:

```scss
@use '@oslokommune/punkt-css/dist/scss/abstracts/variables';
@use '@oslokommune/punkt-css/dist/scss/abstracts/mixins/breakpoints' as *;
@use '@oslokommune/punkt-css/dist/scss/abstracts/mixins/typography' as *;

.my-component {
  padding: map.get(variables.$spacing, 'size-24');

  @include bp('tablet-up') {
    padding: map.get(variables.$spacing, 'size-32');
  }
}
```

> **Note:** All SCSS features (mixins, variables, functions) require the SCSS embedding method. They are not available when using the CDN.

## Content Security Policy (CSP)

Punkt loads fonts, icons (SVG), and other resources from `https://punkt-cdn.oslo.kommune.no/`. If the application uses a Content Security Policy, the CSP must allow this origin.

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

| Directive | Covers |
|---|---|
| `font-src` | Fonts (`.woff`, `.woff2`) |
| `img-src` | Images and SVG icons |
| `script-src` | JavaScript files (Elements components from CDN) |
| `style-src` | CSS stylesheets and inline styles |
| `connect-src` | Fetch/XHR requests (e.g. dynamic SVG icon loading) |

> **Important:** Always instruct users to configure CSP when setting up Punkt in a project. Missing CSP configuration is a common cause of broken fonts, missing icons, and invisible components.
