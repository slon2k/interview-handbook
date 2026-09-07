# Grid

## Definition

CSS Grid is a two-dimensional layout system — it lets rows and columns align with each other simultaneously, which Flexbox can only approximate by nesting multiple flex containers. Grid is the right tool whenever a layout is genuinely a grid: a dashboard, a card gallery, or a page-level layout with header/sidebar/content/footer regions.

```css
.dashboard {
  display: grid;
  grid-template-columns: 250px 1fr;
  grid-template-rows: auto 1fr auto;
  gap: 16px;
}
```

## Alternatives & Trade-offs

Flexbox can fake a grid by wrapping flex items, but items on different rows won't align into consistent columns unless every row happens to have identical content widths — a real constraint Grid expresses directly instead of relying on coincidence. Grid's explicit two-dimensional model costs a small amount of extra syntax (`grid-template-columns`, named areas) compared to Flexbox's simpler mental model, which is exactly why Flexbox remains the better default for genuinely one-dimensional layouts.

## How It Works

### Defining tracks — `fr` units for flexible, proportional space

```css
.layout {
  display: grid;
  grid-template-columns: 200px 1fr 1fr; /* fixed sidebar, then two EQUAL flexible columns */
}
```

```css
.layout {
  grid-template-columns: repeat(3, 1fr); /* three equal columns, shorthand for 1fr 1fr 1fr */
}
```

`fr` (fraction) units divide available space proportionally after any fixed-size tracks are subtracted — `1fr 2fr` gives the second column twice the space of the first, out of whatever space remains.

### Named grid areas — mapping layout regions to readable names

```css
.page {
  display: grid;
  grid-template-columns: 200px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "sidebar header"
    "sidebar content"
    "sidebar footer";
}
.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.content { grid-area: content; }
.footer { grid-area: footer; }
```

Named areas make a layout's overall structure readable directly from the CSS — the `grid-template-areas` declaration visually resembles the actual page layout, which is far easier to reason about than a set of numeric row/column line placements for a complex page structure.

### Placing items by line number, when named areas aren't a fit

```css
.item {
  grid-column: 2 / 4;  /* starts at column line 2, ends at column line 4 — spans two column tracks */
  grid-row: 1 / 3;
}
```

### `auto-fit`/`auto-fill` with `minmax()` — a responsive grid with no media queries at all

```css
.card-gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 16px;
}
```

This produces as many 200px-minimum columns as currently fit the container's width, each stretching to fill any leftover space equally — the number of columns adjusts automatically as the viewport resizes, without writing a single `@media` breakpoint for this specific layout.

### `auto-fit` vs. `auto-fill` — a subtle but real difference

```
auto-fill: keeps empty tracks if there's leftover space and not enough items to fill them —
           existing items DON'T stretch to fill that leftover space.
auto-fit:  collapses empty tracks, letting existing items stretch to fill the leftover space instead.
```

For a card gallery where cards should stretch to fill a partially-empty last row, `auto-fit` is almost always the intended choice; `auto-fill` is rarely what's actually wanted unless empty placeholder tracks are specifically desired.

### Grid vs. Flexbox for alignment across containers

```css
/* Flexbox: each row aligns independently — column widths can drift between rows */
.row { display: flex; }

/* Grid: every row shares the SAME column tracks, guaranteeing alignment across rows */
.grid-container { display: grid; grid-template-columns: repeat(3, 1fr); }
```

## Application

Use Grid for genuinely two-dimensional layouts — page-level structure, dashboards, card galleries needing aligned columns across multiple rows. Use named grid areas for complex, human-readable page structure; use line-based placement for more ad hoc or overlapping item positioning. Use `auto-fit`/`minmax()` for a responsive grid that adapts without explicit breakpoints.

## Common Mistakes

- Using nested Flexbox to fake a grid, then being surprised when column widths drift between rows that have different content.
- Confusing `auto-fit` and `auto-fill`, ending up with unwanted empty tracks (or unwanted stretching) in a responsive card gallery.
- Writing named grid areas that don't form a valid rectangular grid, causing the layout to silently fail or behave unexpectedly.
- Reaching for Grid for a genuinely one-dimensional layout (a simple toolbar) where Flexbox would be simpler and equally correct.

## Common Interview Questions

### Basic
- What's the core difference between Grid and Flexbox?
- What does an `fr` unit represent?

### Intermediate
- How would you build a responsive card gallery that adjusts its column count without writing media queries?
- What's the difference between `auto-fit` and `auto-fill`?

### Advanced
- Why can nested Flexbox rows fail to keep their columns aligned, while Grid guarantees it?
- How would you design a page layout (header, sidebar, content, footer) using named grid areas?

### Follow-up Questions
- Can Grid and Flexbox be used together in the same layout, at different levels of nesting?
- Does `minmax(200px, 1fr)` guarantee a column is never smaller than 200px, even if the container itself is narrower?

### Code Prediction
Given `grid-template-columns: repeat(auto-fit, minmax(200px, 1fr))` inside a 650px-wide container with 16px gaps, roughly how many columns would this produce, and would the resulting columns be exactly 200px or wider?

## Practical Tasks

- Build a page layout (header, sidebar, content, footer) using named grid areas.
- Build a responsive card gallery using `auto-fit` and `minmax()` with no explicit media queries.
- Reproduce the column-misalignment problem from nested Flexbox rows, then fix it by converting to Grid.

## Readiness Criteria

Define grid tracks and named areas correctly, use `auto-fit`/`minmax()` for a responsive grid without media queries, and explain precisely when Grid's two-dimensional alignment guarantee matters over Flexbox.

## References

- [MDN: Basic concepts of grid layout](https://developer.mozilla.org/docs/Web/CSS/CSS_grid_layout/Basic_concepts_of_grid_layout)
- [MDN: Grid template areas](https://developer.mozilla.org/docs/Web/CSS/grid-template-areas)
