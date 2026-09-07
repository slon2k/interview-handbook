# CSS Cascade, Specificity, and Inheritance

## Definition

When multiple CSS rules target the same element and property, the **cascade** decides which one wins, based on origin/importance, then **specificity** (how precisely a selector targets the element), then source order as the final tiebreaker. **Inheritance** is separate: certain properties (mostly text-related) pass down from parent to child automatically unless overridden, while most others (layout properties like `margin`, `border`) don't inherit at all.

```css
p { color: blue; }         /* specificity: 0-0-1 (one element selector) */
.intro { color: green; }    /* specificity: 0-1-0 (one class) — wins over the element selector */
#main p.intro { color: red; } /* specificity: 1-1-1 (one id, one class, one element) — wins over both */
```

## Alternatives & Trade-offs

Relying purely on specificity to control which styles win works but gets fragile fast — as a codebase grows, teams end up in "specificity wars," adding more specific selectors just to override earlier ones, or reaching for `!important` as an escape hatch that then needs an even stronger override later. A deliberate CSS architecture (naming conventions, CSS Modules, utility classes — awareness-level in this module, covered further where relevant in Module 5) keeps specificity flat and predictable by design, avoiding the arms race entirely.

## How It Works

### Calculating specificity — a three-part score

```
Specificity is compared as (ID count, class/attribute/pseudo-class count, element/pseudo-element count),
compared left to right — a single ID beats any number of classes; a single class beats any number
of element selectors.

#header           -> (1, 0, 0)
.nav .link         -> (0, 2, 0)
nav a                -> (0, 0, 2)
#header .nav .link    -> (1, 2, 0)  -- wins over ALL of the above
```

```css
.button { background: blue; }
.button.primary { background: green; } /* (0,2,0) beats (0,1,0) — wins regardless of order */
```

### Source order only matters when specificity is tied

```css
.button { color: blue; }
.button { color: green; } /* SAME specificity as the rule above — this one wins, since it's later */
```

If two rules have identical specificity, the one appearing later in the stylesheet (or later in document order for equivalent `<style>` blocks) wins — this is why reordering CSS rules can silently change which one applies, when neither has a specificity advantage over the other.

### `!important` and inline styles — breaking out of normal specificity entirely

```css
.button { color: blue !important; } /* wins over almost anything except another !important with higher specificity */
```

```html
<div style="color: red;"> <!-- inline styles beat any selector-based rule, !important aside -->
```

`!important` is a signal that specificity is already out of control in a codebase — it solves the immediate problem while making the *next* override even harder, since now an even higher-specificity `!important` rule is needed to beat it.

### Inheritance — text properties cascade down; box/layout properties don't

```css
body { font-family: sans-serif; color: #333; } /* inherited by every descendant automatically */
```

```html
<body> <!-- font-family and color apply here -->
  <p>This text inherits font-family and color from body.</p>
  <div style="border: 1px solid black;">
    <p>This paragraph does NOT inherit the border — border never inherits, regardless of nesting.</p>
  </div>
</body>
```

`color`, `font-family`, `font-size`, and `line-height` are commonly inherited; `margin`, `padding`, `border`, and `background` are not — a `<div>` with a border doesn't pass that border down to its children just because it's a parent.

### Forcing or blocking inheritance explicitly

```css
.reset-box { all: unset; }    /* removes inherited AND cascaded values, resetting to initial */
.force-inherit { border: inherit; } /* explicitly inherits a property that wouldn't inherit by default */
```

## Application

Keep selectors as flat and low-specificity as practical (favor classes over IDs and deeply nested selectors) to avoid specificity wars later. Reserve `!important` for genuine, rare overrides (like overriding a third-party library's inline styles) rather than as a routine tool. Know which properties inherit by default so you're not surprised when a border or margin doesn't pass down, or when a font-family unexpectedly does.

## Common Mistakes

- Reaching for higher specificity (extra classes, an ID selector) to "win" against an earlier rule, instead of addressing why the earlier rule was too specific or too broad in the first place.
- Using `!important` as a routine fix, starting a specificity arms race that gets progressively harder to unwind.
- Assuming all CSS properties inherit, then being confused when a `border` or `padding` set on a parent doesn't appear on its children.
- Forgetting that source order is the tiebreaker only when specificity is genuinely equal — reordering rules with different specificity has no effect on which one wins.

## Common Interview Questions

### Basic
- How is CSS specificity calculated?
- What's the difference between the cascade and inheritance?

### Intermediate
- Why does `!important` tend to make a codebase's CSS harder to maintain over time, even though it solves an immediate problem?
- Which common CSS properties inherit by default, and which don't?

### Advanced
- How would you refactor a stylesheet caught in a "specificity war" (many overrides stacking on top of each other) toward flatter, more maintainable selectors?
- When does source order actually determine which rule wins, versus when does specificity make source order irrelevant?

### Follow-up Questions
- Does an inline `style` attribute always win over any class or ID selector?
- Can inheritance be forced for a property that doesn't inherit by default?

### Code Prediction
Given `.button { color: blue; }` followed later by `.card .button { color: green; }`, and an element with `class="button"` nested inside an element with `class="card"`, which color applies, and why?

## Practical Tasks

- Calculate the specificity of a set of selectors and predict which rule wins for a given element.
- Refactor a stylesheet using `!important` to instead resolve the conflict through selector structure.
- Identify which properties in a given stylesheet rely on inheritance versus which are set explicitly on every element.

## Readiness Criteria

Calculate specificity precisely, explain when source order acts as the tiebreaker, and predict which CSS properties inherit by default versus which require explicit inheritance.

## References

- [MDN: Cascade, specificity, and inheritance](https://developer.mozilla.org/docs/Web/CSS/Cascade)
- [MDN: Specificity](https://developer.mozilla.org/docs/Web/CSS/Specificity)
