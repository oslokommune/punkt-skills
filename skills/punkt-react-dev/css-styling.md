# CSS / Styling

## BEM Structure

All CSS classes follow: `.pkt-{block}__element--modifier`

- **Block**: `pkt-btn`, `pkt-accordion`, `pkt-header-service`
- **Element**: `pkt-btn__text`, `pkt-btn__icon`, `pkt-accordion-item__content`
- **Modifier**: `pkt-btn--primary`, `pkt-btn--small`, `pkt-accordion--compact`
- **State**: `pkt-btn--isLoading`, `is-open`

## CSS source

All SCSS lives in `packages/css/src/scss/`:
- `components/` — higher-level component styles (accordion, alert, card, modal, etc.)
- `elements/` — base element styles (button, input, etc.)
- `abstracts/` — shared variables, mixins, functions, breakpoints

## No inline styles

Do not use inline styles (`style={{ ... }}`) in React components unless explicitly necessary. All styling is handled through BEM classes from `@oslokommune/punkt-css`. When styling changes are needed, update the relevant SCSS in `packages/css/src/scss/` rather than adding inline styles to the React component.

## Prop-to-class mapping

Props translate directly to BEM modifiers:
```tsx
// size="small" → pkt-btn--small
// skin="primary" → pkt-btn--primary
// variant="icon-left" → pkt-btn--icon-left
size && `pkt-btn--${size}`
```
