# Table

Table displays structured data in rows and columns. It can be used as a React compound component or with plain HTML and CSS classes.

## Availability

| Package  | Available | Tag / Import                                                                                                                           |
| -------- | --------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| React    | Yes       | `import { PktTable, PktTableHeader, PktTableHeaderCell, PktTableBody, PktTableRow, PktTableDataCell } from '@oslokommune/punkt-react'` |
| CSS only | Yes       | Apply `pkt-table` classes to standard `<table>` HTML                                                                                   |
| Elements | No        | —                                                                                                                                      |

Dark mode: Yes

## Sub-components (React)

| Component         | React                  | HTML equivalent | CSS class                |
| ----------------- | ---------------------- | --------------- | ------------------------ |
| Table             | `<PktTable>`           | `<table>`       | `pkt-table`              |
| Table Header      | `<PktTableHeader>`     | `<thead>`       | `pkt-table__header`      |
| Table Header Cell | `<PktTableHeaderCell>` | `<th>`          | `pkt-table__header-cell` |
| Table Body        | `<PktTableBody>`       | `<tbody>`       | `pkt-table__body`        |
| Table Row         | `<PktTableRow>`        | `<tr>`          | `pkt-table__row`         |
| Table Data Cell   | `<PktTableDataCell>`   | `<td>`          | `pkt-table__data-cell`   |

## CSS classes

| Class                     | Description                                                         |
| ------------------------- | ------------------------------------------------------------------- |
| `pkt-table`               | Base table class (required)                                         |
| `pkt-table--basic`        | Default skin with bottom borders on rows                            |
| `pkt-table--zebra`        | Alternating gray background on odd rows                             |
| `pkt-table--zebra-blue`   | Deprecated alias for `pkt-table--zebra`                             |
| `pkt-table--compact`      | Reduced padding                                                     |
| `pkt-table--responsive`   | Stacks rows at 768px or narrower (screen or nearest container)      |
| `pkt-table__caption`      | Table title (`<caption>`), flush left, 16px above the table         |
| `pkt-table__header`       | Header section (`<thead>`)                                          |
| `pkt-table__header-cell`  | Header cell (`<th>`)                                                |
| `pkt-table__body`         | Body section (`<tbody>`)                                            |
| `pkt-table__row`          | Table row (`<tr>`)                                                  |
| `pkt-table__data-cell`    | Data cell (`<td>`)                                                  |
| `pkt-table__cell-wrapper` | Lays out several elements in a cell in a row with a gap (e.g. tags) |

## Responsive view

When `pkt-table--responsive` is added to the table, cells stack vertically when the screen is 48rem (768px) or narrower, or when the nearest query container is. A table can't be its own container, so to make it respond to the space it has, give an element around it `container-type: inline-size`. Without a container, only the screen width counts. Each cell shows a label from its `data-label` attribute next to the cell content.

**Requirements for responsive to work:**

1. Add `pkt-table--responsive` class to the `<table>` (or set `responsiveView` prop in React — enabled by default)
2. Every `<td>` must have a `data-label` attribute matching its column header text

On mobile, `<thead>` is hidden and each row becomes a block of label–value pairs, with the label from `data-label` beside the cell's content. A cell keeps its content together, so a cell with running text, links, tags or block elements needs no wrapper. A `<th>` in a body row (a row header, `scope="row"`) is shown as the block's title. Labels are as wide as the longest label in the block, up to 10rem, and wrap beyond that. `pkt-table--compact` keeps 16px at the sides and has 12px between pairs. `pkt-table--basic` draws a line above the first block and between blocks.

## Title

Give a table a title with `<caption>` as the first child of `<table>`. Inside `.pkt-table` it gets `pkt-txt-20-medium`, starts at the table's left edge and has 16px below it, on all screen sizes. The class `pkt-table__caption` gives the same style. In React, put `<caption>` directly inside `PktTable`. The caption is also the table's accessible name.

```html
<table class="pkt-table pkt-table--basic pkt-table--responsive">
  <caption>
    Participants
  </caption>
  <thead class="pkt-table__header">
    ...
  </thead>
</table>
```

## Props / Attributes — PktTable (React)

Types `IPktTable`, `IPktTableDataCell` and `TPktTableSkin` are exported from `@oslokommune/punkt-react`.

| Prop (React)     | Type                   | Default   | Description                          |
| ---------------- | ---------------------- | --------- | ------------------------------------ |
| `skin`           | `"basic"` \| `"zebra"` | `"basic"` | Visual style                         |
| `compact`        | boolean                | `false`   | Compact mode with less padding       |
| `responsiveView` | boolean                | `true`    | Enable responsive stacking on mobile |

`skin="zebra-blue"` and the class `pkt-table--zebra-blue` are deprecated and will be removed in a future version. They still work, render the same as `zebra`, and the prop logs a `console.warn` once. Use `skin="zebra"` / `pkt-table--zebra`.

## Props / Attributes — PktTableDataCell (React)

| Prop (React) | Type   | Default | Description                                                 |
| ------------ | ------ | ------- | ----------------------------------------------------------- |
| `dataLabel`  | string | —       | Label shown on mobile — should match the column header text |

## Props / Attributes — PktTableHeaderCell (React)

Takes all `<th>` attributes. `role` follows `scope`: `scope="row"` or `scope="rowgroup"` gives `role="rowheader"`, anything else `role="columnheader"`. An explicit `role` wins.

```jsx
<PktTableRow>
  <PktTableHeaderCell scope="row">Water</PktTableHeaderCell>
  <PktTableDataCell dataLabel="Price">10 kr</PktTableDataCell>
</PktTableRow>
```

## Accessibility

- Use `<thead>`/`<th>` (or `PktTableHeader`/`PktTableHeaderCell`) for column headings — screen readers use these to announce cell context
- For a heading at the start of a body row, use `scope="row"` so it is announced as a row header
- Add `role="table"` on the table and `role="row"` on rows for explicit semantics

## Examples

### React

```jsx
import {
  PktTable,
  PktTableHeader,
  PktTableHeaderCell,
  PktTableBody,
  PktTableRow,
  PktTableDataCell,
} from '@oslokommune/punkt-react'

{
  /* responsiveView is true by default — dataLabel is needed on each cell */
}
;<PktTable skin="basic">
  <PktTableHeader>
    <PktTableRow>
      <PktTableHeaderCell>Name</PktTableHeaderCell>
      <PktTableHeaderCell>Email</PktTableHeaderCell>
      <PktTableHeaderCell>Role</PktTableHeaderCell>
    </PktTableRow>
  </PktTableHeader>
  <PktTableBody>
    <PktTableRow>
      <PktTableDataCell dataLabel="Name">Ola Nordmann</PktTableDataCell>
      <PktTableDataCell dataLabel="Email">ola@example.com</PktTableDataCell>
      <PktTableDataCell dataLabel="Role">Admin</PktTableDataCell>
    </PktTableRow>
    <PktTableRow>
      <PktTableDataCell dataLabel="Name">Kari Hansen</PktTableDataCell>
      <PktTableDataCell dataLabel="Email">kari@example.com</PktTableDataCell>
      <PktTableDataCell dataLabel="Role">User</PktTableDataCell>
    </PktTableRow>
  </PktTableBody>
</PktTable>
```

### CSS only (HTML)

```html
<table class="pkt-table pkt-table--basic pkt-table--responsive">
  <thead class="pkt-table__header">
    <tr class="pkt-table__row" role="row">
      <th class="pkt-table__header-cell">Date</th>
      <th class="pkt-table__header-cell">Status</th>
      <th class="pkt-table__header-cell">Description</th>
      <th class="pkt-table__header-cell">Reviewed by</th>
    </tr>
  </thead>
  <tbody class="pkt-table__body">
    <tr class="pkt-table__row" role="row">
      <td class="pkt-table__data-cell" data-label="Date">19.01.2023 08:08</td>
      <td class="pkt-table__data-cell" data-label="Status">
        <span class="pkt-table__cell-wrapper">
          <span class="pkt-tag pkt-tag--thin-text pkt-tag--yellow">Normal</span>
        </span>
      </td>
      <td class="pkt-table__data-cell" data-label="Description">Summary of the item</td>
      <td class="pkt-table__data-cell" data-label="Reviewed by">Ola Nordmann</td>
    </tr>
  </tbody>
</table>
```

The BEM classes (`pkt-table__header`, `pkt-table__row`, etc.) are optional — the CSS also targets standard `<thead>`, `<tr>`, `<th>`, `<td>` elements within `.pkt-table`. A minimal CSS-only table can use just the base class:

```html
<table class="pkt-table pkt-table--zebra pkt-table--compact">
  <thead>
    <tr>
      <th>Column A</th>
      <th>Column B</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td data-label="Column A">Value 1</td>
      <td data-label="Column B">Value 2</td>
    </tr>
  </tbody>
</table>
```
