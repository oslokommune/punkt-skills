# Search Input

> **Recommendation:** For most search use cases, use [`<PktTextinput>`](textinput.md) with `type="search"` instead. It provides the same native search affordances (clear button, search icon) with less overhead. Use SearchInput only when you need built-in autocomplete suggestions or the `global` appearance.

Search Input provides a search field with optional autocomplete suggestions. It comes in different appearances for local in-page search and global site-wide search.

## Availability

| Package        | Available | Tag / Import                                                                                       |
| -------------- | --------- | ---------------------------------------------------------------------------------------------------- |
| React          | Yes       | `<PktSearchInput>` — `import { PktSearchInput } from '@oslokommune/punkt-react'`                   |
| Elements       | Yes       | `<pkt-searchinput>` — `import '@oslokommune/punkt-elements/dist/pkt-searchinput.js'`               |
| Elements (CDN) | Yes       | `<script src="https://punkt-cdn.oslo.kommune.no/latest/elements/pkt-searchinput.js" type="module">` |

Dark mode: No

## Variants

| Appearance          | Use                                           |
| ------------------- | --------------------------------------------- |
| `local` (default)   | In-page search, simple field with search icon |
| `local-with-button` | In-page search with a visible search button   |
| `global`            | Full-width search bar for site-wide search    |

## Props / Attributes

| Prop (React)  | Type                                                     | Default        | Description                                      |
| ------------- | -------------------------------------------------------- | -------------- | ------------------------------------------------ |
| `name`        | string                                                   | **(required)** | Form field name                                  |
| `id`          | string                                                   | **(required)** | Unique identifier                                |
| `appearance`  | `"local"` \| `"local-with-button"` \| `"global"`         | `"local"`      | Visual appearance                                |
| `placeholder` | string                                                   | —              | Hint text shown when empty                       |
| `value`       | string                                                   | —              | Controlled value                                 |
| `label`       | string                                                   | —              | Label for the search field                       |
| `disabled`    | boolean                                                  | `false`        | Disables the field                               |
| `fullwidth`   | boolean                                                  | `false`        | Field takes full width                           |
| `inputSize`   | `"xsmall"` \| `"small"` \| `"medium"`                    | `"medium"`     | Field height                                     |
| `action`      | string                                                   | —              | Form action URL                                  |
| `suggestions` | `Array<{ title: string, text?: string, href?: string }>` | `[]`           | Autocomplete suggestions displayed in a dropdown |

In Elements the attributes carry the same names, except `inputSize` which is `input-size`. Elements additionally has a `method` attribute for the form method. Pass `suggestions` as a JavaScript property.

## Events

| Event (React)       | Event (Elements)        | Description                                    |
| ------------------- | ----------------------- | ---------------------------------------------- |
| `onChange`          | —                       | Fires when the search value changes            |
| `onSearch`          | `pkt-search`            | Fires when the user submits the search (Enter) |
| `onSuggestionClick` | `pkt-suggestion-click`  | Fires when a suggestion is clicked             |

Note the `pkt-` prefix on the Elements events here — most Punkt elements emit unprefixed names.

## Examples

### React

```jsx
import { PktSearchInput } from '@oslokommune/punkt-react'

{
  /* Basic local search */
}
;<PktSearchInput
  name="search"
  id="search"
  placeholder="Search..."
  onSearch={(value) => handleSearch(value)}
/>

{
  /* With search button */
}
;<PktSearchInput
  name="search"
  id="search-btn"
  appearance="local-with-button"
  placeholder="Search..."
  onSearch={(value) => handleSearch(value)}
/>

{
  /* With suggestions */
}
;<PktSearchInput
  name="search"
  id="search-suggest"
  placeholder="Search cities..."
  suggestions={[
    { title: 'Oslo', text: 'Capital of Norway' },
    { title: 'Bergen', text: 'Western capital' },
    { title: 'Trondheim', text: 'Technology hub' },
  ]}
  onChange={(value) => updateSuggestions(value)}
  onSuggestionClick={(suggestion) => navigate(suggestion.href)}
/>

{
  /* Global search */
}
;<PktSearchInput
  name="global-search"
  id="global-search"
  appearance="global"
  placeholder="Search the site..."
  fullwidth
/>
```
