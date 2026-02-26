# ESLint

## Configuration

**File:** `packages/elements/eslint.config.js`
**Format:** ESLint 9 flat config

### Plugins

- **eslint-plugin-lit** — Lit web component best practices (flat/recommended)
- **@typescript-eslint** — TypeScript linting
- **eslint-config-prettier** — disables formatting rules that conflict with Prettier

### Lit base class registration

The config registers all Punkt base classes so `eslint-plugin-lit` recognizes them:

```javascript
settings: {
  lit: {
    elementBaseClasses: [
      'PktShadowElement',
      'PktElement',
      'PktInputElement',
      'PktOptionsInputElement',
    ],
  },
}
```

## Rules

### Errors

| Rule | Description |
|---|---|
| `camelcase` | Enforce camelCase naming |
| `no-duplicate-imports` | No duplicate import statements |
| `no-console` | No `console.log` / `console.warn` / `console.error` |
| `no-alert` | No `alert()` / `confirm()` / `prompt()` |

### Warnings

| Rule | Description |
|---|---|
| `@typescript-eslint/no-explicit-any` | Warn on `any` type (use eslint-disable comment when necessary) |

### Disabled

| Rule | Reason |
|---|---|
| `@typescript-eslint/ban-ts-comment` | Allows `@ts-ignore` / `@ts-expect-error` |
| `@typescript-eslint/no-empty-function` | Allows empty functions (useful for no-op handlers) |

### Test file exceptions

For `**/*.test.ts` and `**/test-framework.ts`:
- `no-console: 'off'`
- `@typescript-eslint/no-unused-vars: 'off'`
- `@typescript-eslint/no-explicit-any: 'off'`

### Lit-specific rules (from eslint-plugin-lit recommended)

Key rules enforced:
- `lit/no-invalid-html` — validates HTML template syntax
- `lit/no-legacy-template-syntax` — prevents deprecated template patterns
- `lit/attribute-value-entities` — proper HTML entity escaping in attributes
- `lit/binding-positions` — validates binding locations in templates
- `lit/no-duplicate-template-bindings` — no duplicate bindings on same element
- `lit/no-property-change-update` — don't change reactive properties in `update()`

### Excluded paths

`src/docs/**/*` is excluded from linting (demo/documentation components).

## Commands

```bash
npm run lint        # Lint all source files
npm run lint:fix    # Auto-fix lint issues
```
