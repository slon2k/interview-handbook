# Flexbox

## Definition

Flexbox is a one-dimensional layout system for distributing and aligning items along a row or column. It is well suited to toolbars, form rows, card internals, and other layouts whose primary relationship is one axis.

## How It Works

- A flex container establishes a main axis through `flex-direction` and a perpendicular cross axis.
- Use `justify-content` for main-axis distribution and `align-items` or `align-self` for cross-axis alignment.
- `flex-grow`, `flex-shrink`, and `flex-basis` control how items claim, surrender, and begin with space.
- Flex items have an automatic minimum size that can preserve long content and cause overflow. Use a deliberate `min-width`, often `0`, where a text region must shrink or truncate.
- Use `gap` for the space between items rather than margins that need special first/last-child handling.
- Use `flex-wrap` when independent items may move onto another line; use Grid when tracks must align across both axes.

## Alternatives & Trade-offs

Flexbox is clearer than Grid when the layout is fundamentally a row or a column. It becomes harder to reason about when rows and columns must align with each other, in which case Grid expresses the constraint directly.

## Common Mistakes

- Treating `justify-content` as horizontal alignment regardless of `flex-direction`.
- Forgetting that flex items shrink by default but may have an automatic minimum size.
- Using margins instead of `gap` for ordinary inter-item spacing.
- Using Flexbox for a two-dimensional data or card grid that needs aligned tracks.

## Common Interview Questions

### Foundation

- What are the main and cross axes?
- What is the difference between `justify-content` and `align-items`?
- What do `flex-grow`, `flex-shrink`, and `flex-basis` control?

### Intermediate

- Why might a flex item overflow instead of shrinking as expected?
- How would you make a toolbar wrap cleanly at narrow widths?
- When would you use Flexbox rather than Grid?

### Advanced and Follow-up

- How would you make a flexible text region truncate while adjacent controls retain their size?
- How do intrinsic sizes affect a flex layout?

## Practical Tasks

- Build a toolbar whose search input grows while action buttons retain a usable width.
- Repair a flex row that overflows because of a long unbreakable value.
- Convert margin-based item spacing to `gap`.

## Readiness Criteria

You can choose Flexbox for one-dimensional relationships, predict its sizing behaviour, and diagnose common wrapping and overflow defects.

## References

- [MDN: Flexbox](https://developer.mozilla.org/docs/Learn/CSS/CSS_layout/Flexbox)
- [MDN: Basic concepts of flexbox](https://developer.mozilla.org/docs/Web/CSS/CSS_flexible_box_layout/Basic_concepts_of_flexbox)