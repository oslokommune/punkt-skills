# Punkt Design System Skills

Claude Code plugin with skills for using and developing the [Punkt design system](https://punkt.oslo.kommune.no/) by Oslo kommune.

Disclaimer: This plugin is somewhat experimental. Use it with care, and always check the generated output manually. AI agents are unpredictable, and errors might occur.

## Skills

| Skill                  | Description                                                                                           |
| :--------------------- | :---------------------------------------------------------------------------------------------------- |
| **punkt-components**   | Using Punkt UI components (React and Web Components) — APIs, props, events, slots, and usage patterns |
| **punkt-css**          | Using Punkt CSS — setup, layout, typography, colors, grid, spacing, and utility classes               |
| **punkt-elements-dev** | Developing Lit web components in `@oslokommune/punkt-elements`                                        |
| **punkt-react-dev**    | Developing React components in `@oslokommune/punkt-react`                                             |
| **punkt-css-dev**      | Developing component and utility styles in `@oslokommune/punkt-css`                                   |

## Installation

```
/plugin install punkt-designsystem@dig
```

(Requires access to the Claude marketplace for Digitaliseringsdirektoratet, Oslo kommune)

## Usage

Skills are activated automatically by Claude when your request matches what they cover. For example:

- Ask Claude to build a form with Punkt components and the **punkt-components** skill kicks in
- Work on SCSS in the Punkt CSS source and the **punkt-css-dev** skill provides conventions and patterns
- Create a new Lit web component and the **punkt-elements-dev** skill walks you through the checklist

## Versioning

The plugin version is bumped automatically on push to `main` based on [Conventional Commits](https://www.conventionalcommits.org/).

| Commit prefix | Bump  | Example                                   |
| :------------ | :---- | :---------------------------------------- |
| `feat:`       | minor | `feat: add tooltip component skill`       |
| `fix:`        | patch | `fix: correct accordion example`          |
| `docs:`       | patch | `docs: update README`                     |
| `refactor:`   | patch | `refactor: reorganize css skill sections` |
| `chore:`      | patch | `chore: update metadata`                  |
| `feat!:`      | major | `feat!: rename all skill directories`     |

A `!` after any prefix (e.g. `feat!:`, `fix!:`) triggers a **major** bump.

Commits that don't match a conventional prefix are ignored.

## Maintainers

Team Designsystem, Digitaliseringsetaten, Oslo kommune — [punkt@dig.oslo.kommune.no](mailto:punkt@dig.oslo.kommune.no)
