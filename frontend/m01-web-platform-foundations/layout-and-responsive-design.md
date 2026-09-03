# Layout and Responsive Design

## Definition

CSS layout determines how elements participate in normal flow and how available space is distributed. Responsive design lets the same content adapt to different viewport sizes, input methods, and user preferences without relying on a fixed device list.

## How It Works

- Use normal flow for content that should naturally expand and reflow.
- Use Flexbox for one-dimensional alignment and distribution, and Grid when relationships across rows and columns matter.
- Use absolute or fixed positioning only for elements that genuinely need to be removed from normal flow, such as overlays or anchored UI.
- Use units such as `rem`, `%`, `ch`, and viewport or container units where they express the design constraint. Do not treat pixels as the only responsive tool.
- Use media or container queries to respond to available space and user preferences such as reduced motion.
- Constrain images with intrinsic dimensions and rules such as `max-width: 100%`; choose `object-fit` based on whether cropping is acceptable.
- Diagnose horizontal overflow, layout shifts, and unexpected wrapping with browser layout and performance tools.

## Alternatives & Trade-offs

Flexbox is often clearer for a row or column whose items share one main axis. Grid is clearer when both axes and track alignment matter. Absolute positioning can be precise for overlays but makes content flow and resizing harder to maintain.

Breakpoints based on content are usually more durable than breakpoints based on named devices. A few meaningful layout changes are easier to maintain than many narrow overrides.

## Common Mistakes

- Solving ordinary layout with absolute positioning or arbitrary margins.
- Building a desktop-only layout and patching it with media queries later.
- Using fixed widths that create mobile overflow or clipped focus.
- Forgetting that flex items can shrink or that long content needs a deliberate wrapping strategy.
- Animating layout-heavy properties when a transform or opacity transition would suffice.

## Common Interview Questions

### Foundation

- When would you use Flexbox versus Grid?
- What is normal flow?
- How would you make an image responsive?

### Intermediate

- How would you build a two-column layout that becomes one column without duplicating markup?
- Why might a flex item overflow instead of shrinking as expected?
- How do media queries and container queries differ?

### Advanced and Follow-up

- How would you diagnose a horizontal scrollbar that appears only on mobile?
- How would you prevent a layout shift when images or asynchronous content load?
- How should a responsive animation respond to `prefers-reduced-motion`?

## Practical Tasks

- Build a responsive results page with a filter region, content grid, and empty state using semantic markup.
- Repair a layout that overflows at narrow widths and explain the smallest change that fixes the cause.
- Add responsive image constraints and reduced-motion handling without changing the component's content.

## Readiness Criteria

You can choose a layout model based on the relationship between elements, build a responsive page without duplicated markup, and diagnose overflow, wrapping, image, and layout-shift problems.

## References

- [MDN: CSS layout](https://developer.mozilla.org/docs/Learn/CSS/CSS_layout)
- [MDN: Flexbox](https://developer.mozilla.org/docs/Learn/CSS/CSS_layout/Flexbox)
- [MDN: CSS Grid layout](https://developer.mozilla.org/docs/Learn/CSS/CSS_layout/Grids)
