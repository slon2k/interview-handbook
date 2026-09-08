# CSS Custom Properties

## Definition

Custom properties (informally "CSS variables") let you define a reusable value once and reference it throughout a stylesheet with `var()`. Unlike a Sass/Less variable, which is a build-time text substitution with no runtime existence, a custom property is a real, live value the browser resolves at render time — it participates in the cascade and inheritance exactly like any other CSS property, and can be read or changed from JavaScript.

```css
:root {
  --primary-color: #3366cc;
  --spacing-unit: 8px;
}

.button {
  background: var(--primary-color);
  padding: var(--spacing-unit) calc(var(--spacing-unit) * 2);
}
```

## Alternatives & Trade-offs

A preprocessor variable (Sass's `$primary-color`) is resolved once at build time into plain CSS — simple, but the value is frozen at compile time and can't respond to anything at runtime (a theme toggle, a user preference, a JavaScript-driven value). A custom property is resolved live by the browser, so it can change after the page has loaded — a dark-mode toggle, a dynamically-set width — without recompiling any CSS at all, at the cost of being a newer, slightly less universally supported mechanism than a preprocessor variable (though support is now effectively universal in modern browsers).

## How It Works

### Declaring and scoping — custom properties follow the cascade, not just a flat namespace

```css
:root {
  --text-color: #222; /* declared on :root, so it's available EVERYWHERE via inheritance */
}

.card {
  --text-color: #666; /* re-declared on .card — overrides the root value for .card and its descendants ONLY */
}

.card p {
  color: var(--text-color); /* resolves to #666 here, because .card's more specific declaration wins */
}
```

A custom property isn't a single global value — like any other inherited CSS property, it can be redeclared at any level of the cascade, and descendants see whichever declaration is closest (most specific) to them, exactly like `color` or `font-family` would behave.

### Fallback values — a second argument to `var()`

```css
.badge {
  background: var(--badge-color, gray); /* uses gray if --badge-color was never defined anywhere in scope */
}
```

The fallback only applies if the custom property is genuinely undefined (never declared in any applicable scope) — it does not apply if the property was declared but explicitly set to an invalid value for that context, which instead falls back to the property's own initial value.

### Reading and writing custom properties from JavaScript — the live, runtime advantage

```javascript
const root = document.documentElement;

getComputedStyle(root).getPropertyValue('--primary-color'); // reads the current, resolved value

root.style.setProperty('--primary-color', '#cc3366'); // updates it — EVERY element using
                                                          // var(--primary-color) re-renders immediately,
                                                          // with no page reload and no recompiled CSS
```

This is the mechanism behind most JavaScript-driven theme togglers: flipping one or two custom properties on `:root` cascades the new values to every element referencing them, instead of needing to toggle a class on every individual themed element.

### A practical dark-mode pattern combining custom properties with a media query or a class

```css
:root {
  --bg: white;
  --text: #222;
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg: #111;
    --text: #eee;
  }
}

body {
  background: var(--bg);
  color: var(--text);
}
```

The `body` rule never changes — only the custom property *values* change based on the user's OS preference, which is both simpler to maintain and avoids duplicating every themed rule for light and dark separately.

## Application

Use custom properties for design tokens (colors, spacing units, font sizes) that should be consistent across a stylesheet and potentially adjustable at runtime — theming, dark mode, or a user-configurable setting. Scope a custom property's redeclaration to the smallest element that actually needs a different value, relying on inheritance to make it available further down without needing to redeclare it everywhere.

## Common Mistakes

- Assuming a custom property behaves like a preprocessor variable (resolved once, at build time), rather than a live value resolved by the browser at render time.
- Forgetting that a custom property's fallback in `var(--x, fallback)` only applies when `--x` is undefined, not when it's explicitly set to something invalid.
- Declaring the same custom property redundantly on many individual elements instead of relying on inheritance from a shared ancestor.
- Not realizing custom properties can be read and set from JavaScript, missing a much simpler alternative to toggling many individual classes for a runtime theme change.

## Common Interview Questions

### Basic
- What's the difference between a CSS custom property and a Sass/Less variable?
- What syntax declares and references a custom property?

### Intermediate
- How would you implement a dark-mode toggle using custom properties?
- What does the second argument to `var()` do, and when does it actually apply?

### Advanced
- Why can a custom property be redeclared at different levels of the cascade, and how does that differ from a single global variable?
- How would you update a themed value at runtime from JavaScript, and why is that not possible with a preprocessor variable?

### Follow-up Questions
- Do custom properties inherit the same way `color` or `font-family` does?
- Is a custom property's fallback used if the property is declared but set to an invalid value?

### Code Prediction
```css
:root { --gap: 8px; }
.container { --gap: 16px; }
.item { margin: var(--gap); }
```
Given `<div class="container"><div class="item"></div></div>`, what `margin` value does `.item` actually resolve to, and why?

## Practical Tasks

- Define a small set of design tokens (colors, spacing) as custom properties on `:root` and use them throughout a sample stylesheet.
- Implement a dark-mode toggle using `prefers-color-scheme` and custom properties, with no duplicated per-element rules.
- Read and update a custom property from JavaScript and verify every element referencing it updates without a page reload.

## Readiness Criteria

Explain why custom properties are live and cascade-aware unlike preprocessor variables, use fallback values correctly, and implement a runtime theme change using custom properties updated from JavaScript.

## References

- [MDN: Using CSS custom properties](https://developer.mozilla.org/docs/Web/CSS/Using_CSS_custom_properties)
- [MDN: prefers-color-scheme](https://developer.mozilla.org/docs/Web/CSS/@media/prefers-color-scheme)
