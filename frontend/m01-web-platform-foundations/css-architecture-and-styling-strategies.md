# CSS Architecture and Styling Strategies

## Definition

CSS architecture is the convention a project uses to keep styles discoverable, scoped, and predictable as the interface grows. The browser still applies the same cascade, specificity, inheritance, and layout rules regardless of whether styles live in a global stylesheet, a CSS Module, utility classes, or a CSS-in-JS library.

## Alternatives & Trade-offs

- **Layered global CSS** keeps selectors in stylesheets shared by the application. It is simple and direct, but needs conventions and cascade layers to avoid accidental coupling.
- **CSS Modules** scope generated class names to a file or component. They make accidental selector collisions unlikely while keeping ordinary CSS syntax.
- **Utility classes** compose small, predefined declarations in markup. They make the styling vocabulary consistent and avoid naming selectors, but can make JSX visually dense.
- **CSS-in-JS** colocates dynamic styling logic with component code. It can suit component libraries, but adds a runtime or build-time dependency and should not replace an understanding of CSS itself.

## How It Works

CSS Modules transform a local class name into a generated name that is unlikely to collide with another file:

```css
/* Button.module.css */
.primary { background: royalblue; }
```

```javascript
import styles from "./Button.module.css";

export function Button() {
  return <button className={styles.primary}>Save</button>;
}
```

The class is scoped, but its declarations still participate in the cascade. A scoped class does not make specificity, inheritance, responsive design, or accessibility irrelevant.

## Application

Choose the project’s established styling approach before adding a new one. Keep semantic HTML and native controls regardless of how styles are authored. Use CSS custom properties for shared tokens, `gap` and layout primitives for structure, and low-specificity component styles so later overrides remain deliberate.

## Common Mistakes

- Assuming CSS Modules or CSS-in-JS make the cascade disappear.
- Adding a second styling system for one component without a project-level reason.
- Replacing a native control with a styled `div` when a `<button>` or `<input>` can be styled directly.
- Treating utility classes as an excuse to duplicate values instead of using the design system's spacing, color, and typography tokens.

## Common Interview Questions

### Basic

- What problem do CSS Modules solve?
- Do utility classes change how CSS specificity works?

### Intermediate

- When would you choose scoped component styles over global CSS?
- What trade-offs do utility classes introduce in a component codebase?

### Advanced

- How would you reduce specificity conflicts in a mature application without rewriting every stylesheet?
- Why does styling approach not change the need for semantic HTML and accessible controls?

## Practical Tasks

- Identify the styling approach used by an existing React feature and add a small style without introducing a competing system.
- Refactor a collision-prone global selector into a locally scoped style while preserving the resulting layout.

## Readiness Criteria

You can explain the trade-offs between global CSS, CSS Modules, utility classes, and CSS-in-JS, and you know that all of them still rely on the browser's CSS rules.

## References

- [MDN: CSS modules](https://developer.mozilla.org/docs/Web/CSS/CSS_modules)
- [MDN: CSS cascade](https://developer.mozilla.org/docs/Web/CSS/CSS_cascade)
