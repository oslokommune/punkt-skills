# Component Specs (Source of Truth)

Every component has a JSON spec in `/component-specs/`. These define the complete API contract and drive documentation generation.

**Always read the relevant spec before creating or modifying a component.**

Spec structure:
```json
{
  "name": "pkt-button",
  "react": "PktButton",
  "css-class": "pkt-btn",
  "isElement": true,
  "isPureReact": true,
  "dark-mode": true,
  "props": {
    "size": {
      "name": "Storrelse",
      "description": "Knappens storrelse",
      "type": ["small", "medium", "large"],
      "default": "medium",
      "category": "ui"
    }
  },
  "events": { },
  "slots": { },
  "deprecated": { }
}
```

Prop categories: `ui`, `contents`, `tech`, `accessibility`, `functionality`.
