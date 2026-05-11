# web-components-doctor

ESLint plugin for [Spectrum Web Components](https://opensource.adobe.com/spectrum-web-components/) consumers. Catches accessibility gaps, deprecated APIs, and invalid attribute usage **at lint time** — shifting left what `window.__swc.warn()` and axe catch at runtime.

## Installation

```bash
npm install web-components-doctor --save-dev
```

## Quick Start (ESLint 9+ flat config)

```js
// eslint.config.js
import swc from 'web-components-doctor';

export default [
  swc.configs.recommended,
];
```

Or for strict CI enforcement:

```js
export default [
  swc.configs.strict,
];
```

## Rules

### `swc/accessible-component`

Require accessibility attributes on SWC elements.

```js
// ❌ Bad
html`<sp-action-menu></sp-action-menu>`;
html`<sp-picker></sp-picker>`;

// ✅ Good
html`<sp-action-menu label="More actions"></sp-action-menu>`;
html`<sp-picker aria-label="Select option"></sp-picker>`;
```

| Element | Required (at least one of) |
|---|---|
| `<sp-action-menu>` | `label`, `aria-label`, `aria-labelledby` |
| `<sp-avatar>` | `label`, `is-decorative` |
| `<sp-clear-button>` | `label` |
| `<sp-dialog-wrapper>` | `headline` |
| `<sp-picker>` | `label`, `aria-label`, `aria-labelledby` |
| `<sp-progress-bar>` | `label`, `aria-label`, `aria-labelledby` |
| `<sp-progress-circle>` | `label`, `aria-label`, `aria-labelledby` |
| `<sp-status-light>` | `label`, `aria-label`, `aria-labelledby` |
| `<sp-tabs>` | `accessible-label`, `aria-label`, `aria-labelledby` |

### `swc/no-deprecated`

Flag deprecated attributes and attribute values.

```js
// ❌ Bad
html`<sp-button variant="cta">Click</sp-button>`;
html`<sp-overlay allow-outside-click></sp-overlay>`;

// ✅ Good
html`<sp-button variant="accent">Click</sp-button>`;
html`<sp-overlay></sp-overlay>`;
```

| Deprecated | Replacement |
|---|---|
| `<sp-button variant="cta">` | `variant="accent"` |
| `<sp-button variant="overBackground">` | `static-color="white" treatment="outline"` |
| `<sp-button href="...">` | Use native `<a>` element |
| `<sp-overlay allow-outside-click>` | Remove attribute |
| `<sp-status-light variant="accent">` | `"neutral"` or `"info"` |
| `<sp-status-light disabled>` | Remove attribute |

### `swc/required-attributes`

Enforce presence of configuration attributes.

```js
// ❌ Bad
html`<sp-theme></sp-theme>`;
html`<overlay-trigger>content</overlay-trigger>`;

// ✅ Good
html`<sp-theme color="light" scale="medium" system="spectrum"></sp-theme>`;
html`<overlay-trigger triggered-by="click hover">content</overlay-trigger>`;
```

### `swc/valid-attribute-values`

Catch invalid enum values at lint time.

```js
// ❌ Bad
html`<sp-theme color="lightest"></sp-theme>`;
html`<sp-button variant="danger">Click</sp-button>`;

// ✅ Good
html`<sp-theme color="light"></sp-theme>`;
html`<sp-button variant="accent">Click</sp-button>`;
```

## Architecture

This plugin is **data-driven**. Rules are generated from component descriptors rather than hand-written per component:

```
src/
├── adapters/           # Template syntax parsers (Lit today, JSX/HTML next)
│   └── lit-adapter.ts  # Extracts elements from html`` tagged templates
├── core/
│   ├── types.ts        # Normalized IR (ParsedElement, descriptors)
│   └── rule-factory.ts # Generates ESLint rules from descriptors
├── descriptors/
│   └── components.ts   # Component accessibility/deprecation metadata
├── rules/              # Thin wrappers: factory(descriptors)
└── index.ts            # Plugin entry with recommended/strict configs
```

### Adding a new component

Add an entry to `src/descriptors/components.ts`:

```typescript
'sp-new-component': {
  tagName: 'sp-new-component',
  accessibility: {
    requireOneOf: ['label', 'aria-label', 'aria-labelledby'],
  },
  validAttributeValues: {
    variant: ['primary', 'secondary'],
  },
}
```

No new rule code needed — the rule factory picks it up automatically.

### Adding a new template syntax (e.g. JSX)

Create a new adapter in `src/adapters/` that returns `ParsedElement[]` from the relevant AST node type. The rule logic is template-agnostic.

## Configs

| Config | Severity | Use case |
|---|---|---|
| `swc.configs.recommended` | `warn` | Incremental adoption, developer experience |
| `swc.configs.strict` | `error` | CI enforcement, blocking on violations |

## Dynamic values

The plugin **skips** attributes with dynamic template expressions since they can't be statically verified:

```js
// No warning — value is dynamic
html`<sp-action-menu label=${this.label}></sp-action-menu>`;
```

## Related

- [Spectrum Web Components](https://opensource.adobe.com/spectrum-web-components/) — the component library
- [eslint-plugin-lit-a11y](https://github.com/open-wc/open-wc/tree/master/packages/eslint-plugin-lit-a11y) — generic Lit accessibility rules
- [axe-core](https://github.com/dequelabs/axe-core) — runtime accessibility testing
- [Alex Hayton's eslint-plugin-spectrum-web-components-lit](https://github.com/AlexHayton/eslint-plugin-spectrum-web-components-lit) — the original inspiration

## License

Apache-2.0
