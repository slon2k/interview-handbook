# Box Model, Sizing, and Overflow

## Definition

The CSS box model describes the space an element occupies: content, padding, border, and margin. Sizing and overflow rules determine how that box responds when its content or available space does not fit.

## How It Works

- With the default `content-box` model, declared width and height apply to content; padding and border add to the rendered size.
- With `box-sizing: border-box`, declared width and height include padding and border, which usually makes component dimensions easier to reason about.
- Percentage widths resolve against a containing block. A `width: 100%` child can still overflow if padding, borders, a minimum size, or an unbreakable child adds space.
- Use `min-width`, `max-width`, `min-height`, and `max-height` to express constraints instead of relying on one fixed size.
- Treat `overflow` as a content policy. `hidden` can clip text and keyboard focus; `auto` adds scrolling only when needed.
- Use the browser box-model panel and layout overlays to locate unexpected space or overflow before changing CSS.

## Alternatives & Trade-offs

`border-box` is a useful global baseline, but it does not remove the need to account for margins, minimum sizes, and intrinsic content. Horizontal scrolling is appropriate for genuinely wide content such as data tables; it is usually a symptom of a layout defect for ordinary page content.

## Common Mistakes

- Forgetting that padding and borders add to a `content-box` width.
- Hiding overflow to mask a sizing problem, clipping content or focus.
- Assuming an image, long word, or flex item will shrink without an explicit constraint.
- Setting fixed heights for content whose length or text zoom level can vary.

## Common Interview Questions

### Foundation

- How does the box model work?
- What changes when `box-sizing: border-box` is used?
- What is the difference between `overflow: hidden`, `auto`, and `scroll`?

### Intermediate

- Why might a `width: 100%` element still overflow its container?
- How would you constrain an image without distorting it?
- When is horizontal scrolling acceptable?

### Advanced and Follow-up

- How would you diagnose a horizontal scrollbar that appears only at a narrow viewport?
- How can clipping create an accessibility problem even when the layout appears correct?

## Practical Tasks

- Diagnose a component whose declared width overflows after padding is added.
- Repair a narrow-viewport overflow by addressing its sizing cause rather than hiding it.
- Make a media element responsive while preserving its aspect ratio.

## Readiness Criteria

You can calculate common box-model outcomes, express robust size constraints, and diagnose overflow with browser developer tools.

## References

- [MDN: Introduction to the CSS box model](https://developer.mozilla.org/docs/Web/CSS/CSS_box_model/Introduction_to_the_CSS_box_model)
- [MDN: Sizing items in CSS](https://developer.mozilla.org/docs/Learn/CSS/Building_blocks/Sizing_items_in_CSS)
- [MDN: Overflowing content](https://developer.mozilla.org/docs/Learn/CSS/Building_blocks/Overflowing_content)