# Pagination

Pagination splits long lists and search results into pages and lets the user move between them.

**Status: in development, not released.** The design is in review. The look follows the proposal, but props, markup and strings may change before release.

## Availability

| Package        | Available | Tag / Import                                                                             |
| -------------- | --------- | ---------------------------------------------------------------------------------------- |
| React          | Yes       | `<PktPagination>` — `import { PktPagination } from '@oslokommune/punkt-react'`          |
| Elements       | Yes       | `<pkt-pagination>` — `import '@oslokommune/punkt-elements/dist/pkt-pagination.js'`      |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/20/elements/pkt-pagination.js" type="module">` |

Dark mode: Yes, through the button tokens

## Props / Attributes

| Prop (React)   | Attribute (Elements) | Type                                   | Default | Description |
| -------------- | -------------------- | -------------------------------------- | ------- | ----------- |
| `totalPages`   | `total-pages`        | number                                 | —       | Required. Number of pages. With 0 nothing renders |
| `currentPage`  | `current-page`       | number                                 | `1`     | The page the user is on, from 1. The component moves it itself on select, and follows the value when it changes from outside. Elements reflects it to the attribute |
| `siblingCount` | `sibling-count`      | number                                 | `1`     | Pages shown on each side of the current page when it is in the middle of the list. Near the ends the free slots fill with more pages, so 1 always gives 7 items and 2 gives 9 |
| `hrefTemplate` | `href-template`      | string                                 | —       | Makes the pages links. `{page}` is replaced with the page number, e.g. `?side={page}`. Without it the pages are buttons |
| `hideText`     | `hide-text`          | boolean                                | `false` | Show only the arrows on previous and next. The text stays for screen readers. The text is also hidden when the pagination is narrower than 576px (container query) |
| `outlined`     | `outlined`           | boolean                                | `false` | Show the current page as a ring (secondary button) instead of a filled circle (primary button) |
| `smallView`    | `small-view`         | `'ends' \| 'range'`                    | `'ends'` | What a pagination narrower than 400px shows: first, last and current page (`1 … 10 … 20`), or the five pages around the current one (`8 9 10 11 12`) |
| `align`        | `align`              | `'start' \| 'center' \| 'end'`         | `'start'` | Where the pages sit in the width the pagination has. `center` and `end` add `pkt-pagination--center` / `--end` on the `nav` (`justify-content` on the list); `start` adds no class |
| `renderLink`   | —                    | `(args: IRenderLink) => ReactNode`     | —       | React only: render the links with a router link (Next.js, React Router) |
| `strings`      | `strings` (property) | `{ pagination?: { … } }`               | —       | Override texts, see [Strings](strings.md) |
| `aria-label`, `aria-labelledby` | same | string | `Paginering` | Names the `nav`. React spreads them onto it; Elements passes the host attributes to the inner `nav`. Use it when a page has more than one pagination |

## Events

| Event (React)  | Event (Elements) | Description |
| -------------- | ---------------- | ----------- |
| `onPageChange` | `page-change`    | Fires when the user selects another page. React: `(page, event)`. Elements: `event.detail.page` and `event.detail.originalEvent`. With links, `preventDefault()` stops the navigation. Not fired for the current page or a disabled step |

## Behaviour

- From 400px wide, first and last page, the current page and `siblingCount` pages on each side are shown. Narrower, see below. The rest become `…`. Near the ends the slots `…` would take fill with pages instead: page 1 of 20 shows `1 2 3 4 5 … 20`.
- The number of items is the same on every page, so the controls don't move under the pointer. `…` never stands for a single page.
- Narrower than 400px (container query), the view is the one for `siblingCount` 0 by default (`smallView="ends"`): first, last and current page plus gaps, 5 items, so it fits a phone. `smallView="range"` shows the five pages around the current page instead (`paginationRange`). Below 300px the controls also shrink to the xsmall button size (36px, 14px text, 18px icons), 252px for the default. Both lists are rendered as one (`responsivePaginationItems`): items only in the wide list get `pkt-pagination__item--wide-only`, items only in the narrow list (an extra `…`) get `pkt-pagination__item--narrow-only`, which is hidden until then.
- Previous and next stay on the first and last page with `aria-disabled="true"` (links lose `href`), so focus isn't lost.
- Pages are keyed by page number, so focus stays on the selected page when the list moves.
- The logic lives in `shared-utils/pagination/pages.ts` (`paginationItems`), shared by React, Elements and the Astro template. The class lists come from `shared-utils/pagination/classes.ts` (`paginationLinkClass`).
- The list (`.pkt-pagination__list`) is the container (`container-type: inline-size`), not the `nav`. A zero-height `.pkt-pagination::before` grid with `minmax(0, 40rem)` gives the `nav` a preferred width, like Card, so in a flex row or other shrink-to-fit parent it takes up to 640px and shrinks when needed instead of collapsing to 0. In normal flow it fills the width. Wider content (`siblingCount` 3, or 2 with 1000+ pages) wraps in such parents; raise the limit with `--pkt-pagination-preferred-width` (default `40rem`) or give the pagination a width.

## Markup

```html
<nav class="pkt-pagination" aria-label="Paginering">
  <ul class="pkt-pagination__list">
    <li class="pkt-pagination__item">
      <a class="pkt-btn pkt-btn--small pkt-btn--tertiary pkt-btn--icon-left pkt-pagination__link pkt-pagination__link--previous" href="?side=98">
        <pkt-icon class="pkt-icon pkt-btn__icon" name="chevron-thin-left" aria-hidden="true"></pkt-icon>
        <span class="pkt-btn__text">Forrige</span>
      </a>
    </li>
    <li class="pkt-pagination__item">
      <a class="pkt-btn pkt-btn--small pkt-btn--tertiary pkt-pagination__link" href="?side=1">
        <span class="pkt-sr-only">Gå til side 1</span><span aria-hidden="true">1</span>
      </a>
    </li>
    <li class="pkt-pagination__item" aria-hidden="true">
      <span class="pkt-pagination__ellipsis">…</span>
    </li>
    <li class="pkt-pagination__item">
      <a class="pkt-btn pkt-btn--small pkt-btn--primary pkt-pagination__link pkt-pagination__link--current" aria-current="page" href="?side=99">
        <span class="pkt-sr-only">Side 99</span><span aria-hidden="true">99</span>
      </a>
    </li>
    <!-- … -->
  </ul>
</nav>
```

Every control is a small Punkt button (`pkt-btn pkt-btn--small`): tertiary for pages and steps, primary for the current page, secondary with `outlined`. Next uses `pkt-btn--icon-right`. With `hideText` both steps use `pkt-btn--icon-only` instead. Without `hrefTemplate` the controls are `<button type="button">` with the same classes. A disabled step has `aria-disabled="true"` and, as a link, no `href`.

Forced state classes are the button ones: `pkt-btn--hover`, `pkt-btn--focus`, `pkt-btn--active`.

## Examples

### React — links

```jsx
<PktPagination totalPages={160} currentPage={page} hrefTemplate="?side={page}" />
```

### React — buttons in an app

```jsx
const [page, setPage] = useState(1)

<PktPagination totalPages={20} currentPage={page} onPageChange={(next) => setPage(next)} />
```

### Elements

```html
<pkt-pagination total-pages="160" current-page="99" href-template="?side={page}"></pkt-pagination>

<script>
  document.querySelector('pkt-pagination').addEventListener('page-change', (event) => {
    event.preventDefault()
    router.push(`?side=${event.detail.page}`)
  })
</script>
```
