# Flexbox

## Definition

Flexbox is a one-dimensional layout system for distributing and aligning items along a single axis — a row or a column. It's well suited to toolbars, form rows, navigation bars, and card internals, where the primary layout relationship runs in one direction.

```css
.toolbar {
  display: flex;
  justify-content: space-between; /* main-axis distribution */
  align-items: center;              /* cross-axis alignment */
  gap: 12px;
}
```

## Alternatives & Trade-offs

Flexbox is simpler to reason about than Grid when the layout is fundamentally a row or a column — it handles content-driven sizing and wrapping naturally. It becomes harder to express once rows and columns need to align with each other across multiple flex containers (a true two-dimensional relationship), which is exactly the case Grid is built to handle directly rather than approximate.

## How It Works

### Main axis vs. cross axis — and how `flex-direction` flips them

```css
.row { display: flex; flex-direction: row; }     /* main axis: horizontal, cross axis: vertical */
.column { display: flex; flex-direction: column; } /* main axis: VERTICAL, cross axis: horizontal */
```

```css
.row {
  justify-content: center; /* centers items HORIZONTALLY, because main axis is horizontal */
}
.column {
  justify-content: center; /* centers items VERTICALLY here instead — same property, different axis */
}
```

`justify-content` always targets the main axis and `align-items` always targets the cross axis — but which physical direction that actually means flips entirely based on `flex-direction`, which is a common source of "why isn't this centering the way I expected" confusion.

### `flex-grow`, `flex-shrink`, and `flex-basis` — how items claim and give up space

```css
.sidebar { flex: 0 0 250px; }  /* grow: 0, shrink: 0, basis: 250px — a fixed-width sidebar, never grows or shrinks */
.main-content { flex: 1 1 0; }  /* grow: 1, shrink: 1, basis: 0 — takes up all remaining space */
```

```css
.item-a { flex-grow: 1; }
.item-b { flex-grow: 2; } /* claims TWICE as much of the extra space as item-a, once content is placed */
```

`flex-grow` values are ratios, not absolute sizes — an item with `flex-grow: 2` claims twice as much of the *leftover* space as one with `flex-grow: 1`, after every item's base size is accounted for.

### The flex-item overflow trap — automatic minimum size

```css
.toolbar {
  display: flex;
}
.search-input {
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  /* WITHOUT min-width: 0, this input's default min-width is its content's intrinsic size —
     it can refuse to shrink below that, overflowing the toolbar instead of truncating */
  min-width: 0;
}
```

Flex items have an implicit `min-width: auto` (or `min-height: auto` in a column), which can be their content's natural, unshrinkable size — this is exactly why a long unbroken value inside a flex item can overflow its container even though `flex-shrink` is enabled; an explicit `min-width: 0` overrides that default and allows real shrinking/truncation.

### `gap` instead of margins for spacing between items

```css
.toolbar { display: flex; gap: 12px; } /* even, consistent spacing, no first/last-child special-casing needed */
```

```css
/* The old way, before gap was widely supported — needed extra rules to avoid a trailing margin */
.toolbar > * { margin-right: 12px; }
.toolbar > *:last-child { margin-right: 0; }
```

### `flex-wrap` for independent items that should move to a new line

```css
.tag-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}
```

Use `flex-wrap` when items are independent and can wrap onto additional lines freely; reach for Grid instead once items on different lines need to align into consistent columns.

## Application

Use Flexbox for toolbars, navigation, form rows, and any layout whose primary relationship is a single row or column. Set an explicit `min-width: 0` (or `min-height: 0` for columns) whenever a flex item needs to truncate or shrink below its content's natural size. Use `gap` for spacing between items rather than margins.

## Common Mistakes

- Assuming `justify-content` always means horizontal alignment, missing that it targets the main axis, which flips with `flex-direction`.
- Not setting `min-width: 0` on a flex item that needs to shrink or truncate, and being confused when it overflows instead.
- Using margins with first/last-child exceptions for item spacing instead of `gap`.
- Reaching for Flexbox to build a card grid that needs rows and columns to align with each other, when Grid expresses that constraint directly.

## Common Interview Questions

### Basic
- What are the main axis and cross axis, and how does `flex-direction` affect them?
- What's the difference between `justify-content` and `align-items`?

### Intermediate
- Why might a flex item overflow its container instead of shrinking as `flex-shrink` would suggest?
- How would you make a toolbar's search input grow to fill space while its buttons keep a fixed width?

### Advanced
- How do `flex-grow` ratios actually distribute leftover space among multiple items with different values?
- When would you choose Grid over Flexbox for what initially looks like a simple row of items?

### Follow-up Questions
- Does `flex-shrink: 0` prevent an item from ever getting smaller, regardless of its content?
- Is `gap` supported the same way in both Flexbox and Grid?

### Code Prediction
Given `.item-a { flex-grow: 1; }` and `.item-b { flex-grow: 3; }` inside a flex container with 400px of leftover space after both items' base sizes are placed, how much of that 400px does each item claim?

## Practical Tasks

- Build a toolbar where a search input grows to fill available space while adjacent buttons keep a fixed, readable width.
- Reproduce the flex-item overflow trap with a long, unbreakable value, then fix it with `min-width: 0`.
- Convert a flex layout using margin-based spacing with a last-child exception into one using `gap`.

## Readiness Criteria

Predict main-axis/cross-axis behavior correctly across `flex-direction` values, diagnose and fix the flex-item minimum-size overflow trap, and choose Flexbox versus Grid appropriately based on whether the layout is genuinely one-dimensional.

## References

- [MDN: Basic concepts of flexbox](https://developer.mozilla.org/docs/Web/CSS/CSS_flexible_box_layout/Basic_concepts_of_flexbox)
- [MDN: Flexbox and other layout methods](https://developer.mozilla.org/docs/Web/CSS/CSS_flexible_box_layout/Flexbox_and_other_layout_methods)
