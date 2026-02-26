# ESLint

Config: `packages/react/eslint.config.mjs` (flat config, ESLint 9+).

## Key rules

| Rule                                | Level | Note                                           |
| ----------------------------------- | ----- | ---------------------------------------------- |
| `camelcase`                         | error |                                                |
| `no-duplicate-imports`              | error |                                                |
| `no-console`                        | error | Off in test files                              |
| `no-alert`                          | error |                                                |
| `react/react-in-jsx-scope`          | off   | React 17+ auto-import                          |
| `react/prop-types`                  | off   | TypeScript used instead                        |
| `react/display-name`                | off   | Set `displayName` manually when using forwardRef |
| `react-hooks/exhaustive-deps`       | off   |                                                |
| `@typescript-eslint/ban-ts-comment` | off   |                                                |
| `no-restricted-syntax`              | warn  | Forbids `import React` and `import * as React` |

## Test file overrides

Files matching `**/*.test.tsx`, `**/*.test.ts`, `**/setupTests.ts`, `**/test-utils.tsx` relax:

- `no-console` → off
- `@typescript-eslint/no-unused-vars` → off
- `@typescript-eslint/no-explicit-any` → off

## Prettier

Config at `packages/react/.prettierrc`:

```json
{
  "printWidth": 120,
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "all"
}
```
