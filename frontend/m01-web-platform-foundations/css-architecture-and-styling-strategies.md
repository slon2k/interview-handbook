# CSS Architecture and Styling Strategies

## Definition

CSS architecture is the convention a project uses to keep styles discoverable, scoped, and predictable as the interface grows. The browser still applies the same cascade, specificity, inheritance, and layout rules regardless of whether styles live in a global stylesheet, a CSS Module, utility classes, or a CSS-in-JS library — none of these approaches change the underlying rules from earlier in this module, only how class names and file organization are managed on top of them.

```css
/* Button.module.css */
.primary { background: royalblue; }
```

```html
<button class="rounded-lg bg-blue-600 px-4 py-2 text-white">Save</button>
```

## Alternatives & Trade-offs

- **Layered global CSS** keeps selectors in stylesheets shared by the application. It is simple and direct, but needs conventions and cascade layers to avoid accidental coupling, and offers no built-in protection against two unrelated files defining the same class name.
- **CSS Modules** scope generated class names to a file or component. They make accidental selector collisions unlikely while keeping ordinary CSS syntax and full access to the cascade, at the cost of a build step and a layer of indirection between the class name in markup and the name actually generated.
- **Utility classes** (Tailwind and similar) compose small, predefined declarations directly in markup. They make the styling vocabulary consistent and avoid naming selectors entirely, but can make JSX visually dense and shift visual detail into markup rather than a dedicated stylesheet.
- **CSS-in-JS** colocates dynamic styling logic with component code. It can suit component libraries and styles that genuinely depend on runtime values, but adds a runtime or build-time dependency and should not replace an understanding of CSS itself.

## How It Works

### CSS Modules — scoped class names, ordinary CSS syntax

```css
/* Button.module.css */
.primary { background: royalblue; padding: 8px 16px; }
```

```javascript
import styles from "./Button.module.css";

export function Button() {
  return <button className={styles.primary}>Save</button>;
  // styles.primary resolves to something like "Button_primary__a1b2c" at build time —
  // a DIFFERENT file's .primary class can never collide with this one
}
```

### Utility classes — composing a fixed vocabulary directly in markup

```html
<button class="rounded-lg bg-blue-600 px-4 py-2 text-white hover:bg-blue-700">Save</button>
```

There's no separate stylesheet to open at all — every visual property is expressed as a class already defined by the utility framework, trading "naming things" for "composing from a fixed vocabulary" once that vocabulary is familiar.

### A scoped class name still participates in the cascade — scoping and specificity are different concerns

```css
/* Button.module.css */
.primary { color: white; }
```

```css
/* some-other-global-stylesheet.css, loaded LATER in the page */
button { color: black !important; }
```

```javascript
<button className={styles.primary}>Save</button>
// renders BLACK text, not white — the generated class name is unique and collision-free,
// but it's still just an ordinary CSS class, and a later, higher-specificity or !important
// rule from ANYWHERE else on the page still wins, exactly per Module 1's cascade rules
```

Scoping solves *name collisions* — two files can each have a `.primary` class without interfering with each other's selector — but it does nothing to protect a component's styles from being overridden by a more specific or more important rule loaded elsewhere on the page. Cascade layers (`@layer`) can additionally control *ordering* between different styling sources deliberately, which naming scope alone cannot.

## Application

Choose the project's established styling approach before adding a new one — introducing a second system for a single component adds a maintenance burden with no corresponding benefit. Keep semantic HTML and native controls regardless of how styles are authored; a styling approach never changes what Module 1's other topics already require. Use CSS custom properties for shared tokens (colors, spacing) so a theme change doesn't require touching every individual file, and keep component styles at low specificity so later, deliberate overrides remain possible without a specificity fight.

## Common Mistakes

- Assuming CSS Modules or CSS-in-JS make the cascade disappear, when a scoped class name is still just an ordinary class competing on specificity like any other.
- Adding a second styling system for one component without a project-level reason, fragmenting how the codebase's styles are actually organized.
- Replacing a native control with a styled `div` when a `<button>` or `<input>` could be styled directly, losing the native behavior Module 1's earlier topics cover.
- Treating utility classes as an excuse to duplicate literal values (a specific color, a specific spacing number) instead of using the design system's existing tokens.
- Writing global, high-specificity overrides that unexpectedly defeat a scoped component's own styles, then "fixing" it with an escalating `!important`.

## Common Interview Questions

### Basic

- What problem do CSS Modules solve?
- Do utility classes change how CSS specificity works?

### Intermediate

- When would you choose scoped component styles over global CSS?
- What trade-offs do utility classes introduce in a component codebase?

### Advanced

- How would you reduce specificity conflicts in a mature application without rewriting every stylesheet?
- Why does a scoped CSS Modules class remain vulnerable to a later, higher-specificity global rule?

### Follow-up Questions

- Does scoping a class name protect it from being overridden by a more specific selector loaded elsewhere?
- Can cascade layers and CSS Modules be used together, or do they solve the same problem twice?

### Code Prediction

```javascript
import styles from "./Button.module.css"; // .primary { color: white; }
// elsewhere, loaded later: button { color: black !important; }
<button className={styles.primary}>Save</button>
```

What color does this button's text actually render, given the later global rule? What does this reveal about what CSS Modules scoping does and doesn't protect against?

## Practical Tasks

- Identify the styling approach used by an existing React feature and add a small style without introducing a competing system.
- Refactor a collision-prone global selector into a locally scoped CSS Modules class while preserving the resulting layout.
- Reproduce a scoped component style being overridden by a later global rule, then resolve it using cascade layers or by adjusting specificity rather than reaching for `!important`.

## Readiness Criteria

Explain the trade-offs between global CSS, CSS Modules, utility classes, and CSS-in-JS, and demonstrate — not just state — that scoping a class name protects against naming collisions but not against being outranked by a higher-specificity or later-loaded rule.

## References

- [MDN: CSS modules](https://developer.mozilla.org/docs/Web/CSS/CSS_modules)
- [MDN: CSS cascade](https://developer.mozilla.org/docs/Web/CSS/CSS_cascade)
- [MDN: @layer](https://developer.mozilla.org/docs/Web/CSS/@layer)
