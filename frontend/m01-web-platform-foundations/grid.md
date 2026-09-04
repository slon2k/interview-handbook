# Grid

## Definition

CSS Grid is a two-dimensional layout system that defines rows and columns together. It is suited to page regions, card galleries, and layouts where items must align across both axes.

## How It Works

- Define tracks with `grid-template-columns` and `grid-template-rows`; use `fr`, `minmax()`, and intrinsic sizing keywords to describe available space.
- Use `gap` for consistent space between tracks.
- Place items by line number, named area, or automatic placement. Prefer source order that remains meaningful without visual placement.
- `repeat(auto-fit, minmax(...))` can create a responsive card grid whose column count follows available space.
- Grid and Flexbox can be nested: Grid may structure a page while Flexbox aligns controls inside a region.
- Inspect the browser's Grid overlay to see tracks, lines, and implicit rows created by content.

## Alternatives & Trade-offs

Use Grid when both rows and columns describe the design. Use Flexbox when one dimension is primary. Explicit placement can create polished arrangements but should not create a visual order that conflicts with keyboard and reading order.

## Common Mistakes

- Using Grid only because it is newer when a simple flex row communicates the layout better.
- Relying on fixed tracks that fail with translated text or zoom.
- Reordering content visually in a way that makes keyboard navigation confusing.
- Forgetting that implicit tracks can be created when content exceeds the explicit grid.

## Common Interview Questions

### Foundation

- When would you use Grid instead of Flexbox?
- What does the `fr` unit represent?
- What is the difference between explicit and implicit grid tracks?

### Intermediate

- How would you create a card grid that adapts its number of columns to available width?
- When are named grid areas clearer than line numbers?
- How should source order influence Grid placement?

### Advanced and Follow-up

- How would you debug an item appearing in an unexpected implicit row?
- How do you keep a Grid layout robust under zoom and text expansion?

## Practical Tasks

- Build a responsive results grid using `repeat`, `minmax`, and `gap`.
- Create a desktop page-region layout that becomes a single-column flow at narrow widths.
- Use Grid developer tools to identify an implicit track.

## Readiness Criteria

You can use Grid to express two-dimensional constraints, preserve meaningful source order, and build responsive tracks without device-specific column counts.

## References

- [MDN: CSS Grid layout](https://developer.mozilla.org/docs/Learn/CSS/CSS_layout/Grids)
- [MDN: Basic concepts of grid layout](https://developer.mozilla.org/docs/Web/CSS/CSS_grid_layout/Basic_concepts_of_grid_layout)