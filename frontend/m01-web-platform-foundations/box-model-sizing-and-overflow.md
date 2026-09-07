# Box Model, Sizing, and Overflow

## Definition

Every rendered element is a rectangular box composed of content, padding, border, and margin, nested outward in that order. `box-sizing` controls whether a declared `width`/`height` includes padding and border or excludes them — the single most consequential, easy-to-get-wrong sizing decision in CSS. `overflow` controls what happens when content doesn't fit its box.

```css
.card {
  width: 200px;
  padding: 20px;
  border: 5px solid black;
  /* content-box (default): total rendered width = 200 + 20*2 + 5*2 = 250px, NOT 200px */
}
```

## Alternatives & Trade-offs

`box-sizing: content-box` (the default) is how CSS has always worked, but means adding padding or a border to an element with a fixed width silently grows its total rendered size — a common source of unexpected layout breakage. `box-sizing: border-box` makes the declared `width` the *total* size including padding and border, which is far more intuitive and is why virtually every modern reset applies it globally — but it does mean padding changes shrink the available content area instead of growing the box, which is worth being aware of.

## How It Works

### `content-box` vs. `border-box` — the same CSS, two different rendered widths

```css
.content-box { box-sizing: content-box; width: 200px; padding: 20px; border: 5px solid; }
/* rendered width: 200 (content) + 40 (padding) + 10 (border) = 250px */

.border-box { box-sizing: border-box; width: 200px; padding: 20px; border: 5px solid; }
/* rendered width: 200px total — padding and border are subtracted FROM the 200px, not added to it */
```

### The universal `border-box` reset — nearly every modern project does this

```css
*, *::before, *::after {
  box-sizing: border-box;
}
```

With this reset in place, a `width: 200px` element always renders at exactly 200px regardless of how much padding or border is added later — this is why almost every CSS reset/framework applies it globally as one of the very first rules.

### Margin collapsing — vertical margins between siblings don't simply add

```css
.first { margin-bottom: 20px; }
.second { margin-top: 30px; }
/* the GAP between them is 30px (the larger of the two), NOT 50px (their sum) */
```

Adjacent vertical margins collapse into a single margin equal to the larger of the two — a frequent source of "why isn't this spacing what I expected" confusion, and one reason `gap` (on a flex or grid container) is often preferred over margins for spacing between items, since `gap` never collapses.

### `overflow` — what happens when content doesn't fit

```css
.box { width: 200px; height: 100px; overflow: visible; }  /* default: content spills outside the box, unclipped */
.box { overflow: hidden; }   /* content is clipped at the box edge, no scrollbar */
.box { overflow: auto; }      /* a scrollbar appears ONLY if content actually overflows */
.box { overflow: scroll; }     /* a scrollbar is always shown, even if content fits */
```

`overflow: hidden` is also commonly used deliberately to create a "clipping context" — for example, to prevent a child's border-radius corners from being visually overridden by its own content.

### A common overflow trap: `min-width`/`min-height` defaulting to `auto`

```css
.flex-item {
  overflow: hidden;
  /* in a flex or grid container, a flex item's default min-width is `auto`, which can be its
     content's intrinsic size — this can PREVENT overflow:hidden from working as expected,
     unless min-width: 0 is set explicitly */
  min-width: 0;
}
```

This is exactly the flex-item-overflow trap Module 1's Flexbox topic covers from the layout side; the underlying cause is this box-model default.

## Application

Apply a global `border-box` reset at the start of every project so declared widths behave predictably regardless of padding/border changes later. Use `gap` instead of margins for spacing between flex/grid items specifically to avoid margin-collapsing surprises. Set `overflow` deliberately based on whether content should clip, scroll, or spill.

## Common Mistakes

- Forgetting `box-sizing: border-box`, then being surprised that adding padding to a fixed-width element makes it visually larger than intended.
- Expecting adjacent vertical margins to add together, when they actually collapse to the larger of the two.
- Using margins for spacing between flex/grid items and hitting unexpected collapsing behavior, instead of using `gap`.
- Setting `overflow: hidden` on a flex item and being confused when content still overflows, missing the `min-width: auto` default that needs an explicit `min-width: 0` override.

## Common Interview Questions

### Basic
- What's the difference between `content-box` and `border-box`?
- What does `overflow: hidden` do differently from `overflow: auto`?

### Intermediate
- Why does almost every modern CSS reset set `box-sizing: border-box` globally?
- What is margin collapsing, and when does it occur?

### Advanced
- Why might `overflow: hidden` fail to clip content inside a flex item, and what fixes it?
- How would you explain, with a concrete example, why two stacked elements with `margin-bottom: 20px` and `margin-top: 30px` end up with a 30px gap, not 50px?

### Follow-up Questions
- Does margin collapsing apply to horizontal margins as well as vertical ones?
- Does `gap` ever collapse the way adjacent margins do?

### Code Prediction
Given `.box { width: 200px; padding: 20px; border: 5px solid; box-sizing: content-box; }`, what is the total rendered width of this element? What would it be if `box-sizing: border-box` were used instead?

## Practical Tasks

- Apply a global `border-box` reset to a page using `content-box` sizing and observe the layout differences.
- Reproduce margin collapsing between two stacked elements, then eliminate it by switching to a flex container with `gap`.
- Fix a flex item where `overflow: hidden` isn't clipping content as expected, using `min-width: 0`.

## Readiness Criteria

Explain and predict `content-box` versus `border-box` sizing, recognize margin collapsing and its trigger conditions, and diagnose the `min-width: auto` overflow trap in flex layouts.

## References

- [MDN: The box model](https://developer.mozilla.org/docs/Learn/CSS/Building_blocks/The_box_model)
- [MDN: box-sizing](https://developer.mozilla.org/docs/Web/CSS/box-sizing)
- [MDN: Mastering margin collapsing](https://developer.mozilla.org/docs/Web/CSS/CSS_box_model/Mastering_margin_collapsing)
