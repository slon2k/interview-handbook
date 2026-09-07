# Normal Flow and Positioning

## Definition

**Normal flow** is how elements lay out by default with no positioning applied — block elements stack vertically, inline elements flow horizontally within a line. The `position` property lets an element opt out of normal flow entirely (`absolute`, `fixed`) or shift while still reserving its original space (`relative`), each establishing different, easy-to-confuse behavior around what an element is positioned *relative to*.

```css
.static { position: static; }     /* default: normal flow, top/left/etc. have no effect */
.relative { position: relative; top: 10px; } /* shifts visually, but ORIGINAL space is still reserved */
.absolute { position: absolute; top: 0; }     /* removed from flow entirely; positioned relative to nearest positioned ancestor */
.fixed { position: fixed; top: 0; }             /* removed from flow; positioned relative to the VIEWPORT, ignores scrolling */
```

## Alternatives & Trade-offs

Normal flow requires no special properties and is the simplest, most maintainable way to lay out most content — it also naturally adapts to different content lengths and viewport sizes. Positioning (`absolute`/`fixed`) is necessary for overlays, tooltips, and sticky headers that must break out of the normal document flow, but removes the element from the layout entirely, requiring the developer to explicitly manage its size and position rather than letting the flow handle it.

## How It Works

### Block vs. inline in normal flow

```css
div, p, section { display: block; }   /* stacks vertically, takes full available width by default */
span, a, strong { display: inline; }    /* flows horizontally within a line, ignores width/height */
```

```html
<p>This is <span>inline text</span> flowing within the paragraph's line.</p>
<div>This div starts on its own new line, regardless of surrounding content.</div>
```

### `position: relative` — shifts visually, but reserves its original space

```css
.badge {
  position: relative;
  top: -5px; /* moves UP 5px visually */
}
```

```
Before shifting: [ Badge ] takes up a rectangular space in the flow.
After shifting:  the space it ORIGINALLY occupied is still reserved (nothing else moves into
                 it), but the badge itself is drawn 5px higher than that reserved space.
```

This is the detail that surprises people first learning positioning: `position: relative` doesn't remove the element from the flow the way `absolute` does — it just visually offsets it from where it would otherwise sit, leaving a "hole" behind.

### `position: absolute` — removed from flow, positioned relative to the nearest *positioned* ancestor

```css
.tooltip-container {
  position: relative; /* this becomes the "containing block" for any absolutely-positioned children */
}
.tooltip {
  position: absolute;
  top: 100%;
  left: 0;
}
```

An absolutely-positioned element positions itself relative to its nearest ancestor that has *any* `position` value other than `static` (commonly `relative`) — if no ancestor is positioned, it falls back to positioning relative to the initial containing block (roughly, the whole page). This is why `position: relative` is so often added to a parent purely to "anchor" an absolutely-positioned child, even when the parent itself doesn't need to shift anywhere.

### `position: fixed` — relative to the viewport, ignores scrolling

```css
.sticky-header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
}
```

A fixed element stays in the same visual position even as the page scrolls — useful for a persistent header or a "back to top" button, but it's removed from flow just like `absolute`, so content beneath it needs its own top padding/margin to avoid being visually covered.

### `position: sticky` — a hybrid: normal flow until a scroll threshold, then fixed

```css
.section-header {
  position: sticky;
  top: 0; /* sticks to the top of the viewport once scrolled to that point, within its own containing block */
}
```

`sticky` behaves like normal flow until the element would scroll past the specified threshold, at which point it "sticks" — but only within the bounds of its own parent; it un-sticks again once its parent scrolls out of view, unlike `fixed`, which stays anchored to the viewport regardless of any parent.

## Application

Rely on normal flow for the large majority of ordinary content layout. Use `position: relative` primarily to establish a positioning context for an absolutely-positioned child, or for small self-contained visual nudges. Use `absolute`/`fixed` deliberately for overlays, tooltips, and persistent headers, remembering that removing an element from flow means its surrounding layout no longer accounts for its space. Use `sticky` for section headers that should track scroll position within a bounded container.

## Common Mistakes

- Expecting `position: relative` to remove an element from the document flow, when it only visually offsets it while still reserving its original space.
- Forgetting to add `position: relative` to a container meant to anchor an absolutely-positioned child, causing the child to position relative to the page instead of the intended parent.
- Using `position: fixed` for a header without adding corresponding padding to the content below, causing the fixed header to visually cover the first bit of scrollable content.
- Expecting `position: sticky` to behave exactly like `fixed`, missing that sticky elements un-stick once their containing parent scrolls out of view.

## Common Interview Questions

### Basic
- What's the difference between `position: static` and `position: relative`?
- What does "normal flow" mean for block versus inline elements?

### Intermediate
- Why does an absolutely-positioned element sometimes end up positioned relative to the whole page instead of its intended parent?
- What's the practical difference between `position: fixed` and `position: sticky`?

### Advanced
- How would you build a tooltip that positions itself relative to its trigger button, regardless of where that button appears on the page?
- Why does `position: relative` with no `top`/`left`/etc. values still matter for an absolutely-positioned child, even though nothing visually moves?

### Follow-up Questions
- Does `position: absolute` respect the normal flow's block/inline distinction?
- Can a `position: sticky` element stick to the bottom of the viewport instead of the top?

### Code Prediction
Given a `.tooltip-container` with `position: relative` (but no offsets) and a `.tooltip` child with `position: absolute; top: 100%; left: 0;`, relative to what element is the tooltip actually positioned, and why does the parent's `position: relative` matter here even without any `top`/`left` values on the parent itself?

## Practical Tasks

- Build a tooltip that correctly anchors to its trigger element using `position: relative` on the parent and `position: absolute` on the tooltip.
- Reproduce the "absolutely positioned relative to the whole page" bug by omitting a positioned ancestor, then fix it.
- Build a sticky section header that sticks while scrolling through its own section, then un-sticks once that section ends.

## Readiness Criteria

Explain each position value's effect on document flow and its containing-block behavior precisely, and correctly anchor absolutely-positioned elements to their intended parent.

## References

- [MDN: Normal Flow](https://developer.mozilla.org/docs/Web/CSS/CSS_display/Normal_Flow)
- [MDN: position](https://developer.mozilla.org/docs/Web/CSS/position)
