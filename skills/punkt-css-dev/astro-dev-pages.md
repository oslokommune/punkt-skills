# Astro Dev / Test Pages

The CSS package includes an Astro-based dev site for visually testing and demonstrating components. Pages live in `packages/css/src/pages/`.

## Running the dev site

```bash
# From packages/css/
npm run dev        # Start Astro dev server
npm run build-app  # Build static site to dist-app/
npm run preview    # Preview built site
```

## Page structure

Every page follows this template:

```astro
---
import Layout from '../layouts/Layout.astro'
---

<Layout title="Component Name - Punkt CSS">
  <main class="page-main pkt-container pkt-container--laptop">
    <h1>Component Name</h1>

    <section class="component">
      <h2>Variant Name</h2>
      <div class="component__example">
        <!-- Inline component demos -->
      </div>
    </section>

    <section class="component">
      <h2>Another Variant</h2>
      <div class="component__example">
        <!-- More demos -->
      </div>
    </section>
  </main>
</Layout>
```

### Key elements

| Class | Purpose |
|---|---|
| `.page-main` | Page content wrapper |
| `.pkt-container` | Centered layout container (add `--laptop` or `--tablet` for width) |
| `.component` | White background section with padding (dark mode aware) |
| `.component__example` | Flexbox wrapper — items display inline with gap |
| `.component__block-example` | Block layout — items stack vertically |

### Frontmatter

Only the Layout import is needed. No other frontmatter fields:

```astro
---
import Layout from '../layouts/Layout.astro'
---
```

The `title` prop is passed to `<Layout>` and sets the page `<title>`.

## Layouts

### `Layout.astro` (main)

The standard layout. Provides:
- Full HTML document shell with meta tags
- Global CSS via `../main.scss` (imports all of `pkt.scss`)
- Dev header with Oslo kommune logo
- Dark mode toggle (checkbox that sets `data-mode="dark"` on `<body>`)
- Component navigation dropdown (`<select>` listing all pages)
- Inline SVG sprite with common icons
- `<slot />` for page content

### `FocusLayout.astro` (minimal)

Bare layout with no header or navigation. Used for isolated component views:

```astro
---
---
<html>
  <head>
    <style>@import '../scss/pkt-docs.scss';</style>
  </head>
  <body><slot /></body>
</html>
```

## Demonstrating component variants

### Skins / modifiers

Show each skin in its own section with a heading:

```astro
<section class="component">
  <h2>Skin: primary</h2>
  <div class="component__example">
    <button class="pkt-btn pkt-btn--primary">Primary</button>
  </div>
</section>

<section class="component">
  <h2>Skin: secondary</h2>
  <div class="component__example">
    <button class="pkt-btn pkt-btn--secondary">Secondary</button>
  </div>
</section>
```

### Sizes

Group sizes together in one example:

```astro
<section class="component">
  <h2>Sizes</h2>
  <div class="component__example">
    <button class="pkt-btn pkt-btn--small">Small</button>
    <button class="pkt-btn">Medium (default)</button>
    <button class="pkt-btn pkt-btn--large">Large</button>
  </div>
</section>
```

### States

Demonstrate all interactive states using BEM state modifier classes:

```astro
<section class="component">
  <h2>States</h2>
  <div class="component__example">
    <button class="pkt-btn">Normal</button>
    <button class="pkt-btn pkt-btn--hover">Hover</button>
    <button class="pkt-btn pkt-btn--focus">Focus</button>
    <button class="pkt-btn pkt-btn--active">Active</button>
    <button class="pkt-btn" disabled>Disabled</button>
  </div>
</section>
```

### Combined modifiers

Show how modifiers compose:

```astro
<section class="component">
  <h2>Secondary + Small + Icon Left</h2>
  <div class="component__example">
    <button class="pkt-btn pkt-btn--secondary pkt-btn--small pkt-btn--icon-left">
      <span class="pkt-icon pkt-btn__icon">
        <svg viewBox="0 0 32 32"><use href="#user"></use></svg>
      </span>
      <span class="pkt-btn__text">Label</span>
    </button>
  </div>
</section>
```

### Colors

For components with color variants, show all colors in a grid:

```astro
<section class="component">
  <h2>Colors</h2>
  <div class="component__example">
    <button class="pkt-btn pkt-btn--blue">Blue</button>
    <button class="pkt-btn pkt-btn--green">Green</button>
    <button class="pkt-btn pkt-btn--yellow">Yellow</button>
    <button class="pkt-btn pkt-btn--red">Red</button>
  </div>
</section>
```

## Icons

Icons are referenced from the inline SVG sprite defined in `Layout.astro`:

```html
<span class="pkt-icon pkt-btn__icon">
  <svg viewBox="0 0 32 32">
    <use href="#icon-name"></use>
  </svg>
</span>
```

Available icons in the sprite include: `oslologo`, `alert-error`, `alert-information`, `alert-success`, `alert-warning`, `chevron-thin-down`, `chevron-thin-up`, `user`, `menu`, `close`, `cogwheel`, `heart`, `search`, `spinner`, and more.

## JavaScript for interactive components

Most pages are pure CSS demos with no JavaScript. When a component requires JS for interaction (e.g., tabs), use an Astro `<script>` tag:

```astro
<script>
  import PktTabs from '../scripts/pkt-tabs.js'

  const tabList = new PktTabs(
    document.getElementById('PktTabs1'),
    document.getElementById('PktTabs1Panels')
  )

  document.querySelectorAll('#PktTabs1 [role=tab]').forEach((tab, index) => {
    tab.addEventListener('click', () => tabList.setActiveByIndex(index))
  })
</script>
```

## Adding a new page

1. Create `src/pages/{component-name}.astro`.
2. Import `Layout` and wrap content in `<Layout title="...">`.
3. Use `<main class="page-main pkt-container pkt-container--laptop">`.
4. Add sections with `.component` / `.component__example` for each variant group.
5. Demonstrate: default, all skins, all sizes, all states, combined modifiers.
6. Add the page to the navigation dropdown in `src/layouts/Layout.astro` (the `<select>` element).

## Navigation

The dev header in `Layout.astro` contains a `<select>` dropdown that navigates between pages:

```html
<select class="pkt-input" name="dev-header__select" id="dev-header__select">
  <option value="/" selected>Choose component</option>
  <option value="/alert">Alert</option>
  <option value="/button">Button</option>
  <!-- Add new pages here -->
</select>
```

JavaScript in the layout handles navigation on change and highlights the current page.
