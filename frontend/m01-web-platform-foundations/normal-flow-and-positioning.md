# Normal Flow and Positioning

## Definition

Normal flow is the browser's default layout algorithm: block-level content forms a vertical flow and inline content flows within lines. Positioning changes an element's relationship to that flow and, for positioned elements, to a containing block.

## How It Works

- Start with normal flow for content that should expand and reflow as text, localization, and viewport space change.
- `position: relative` keeps an element in flow while allowing it to become a containing block for absolutely positioned descendants.
- `position: absolute` removes an element from normal flow and positions it against its containing block. Use it for anchored UI and overlays, not ordinary page layout.
- `position: fixed` is positioned relative to the viewport unless an ancestor establishes a containing block; it is useful for persistent viewport UI.
- `position: sticky` participates in normal flow until it reaches an inset threshold, then sticks within its scroll container.
- A stacking context isolates `z-index` comparisons. A large `z-index` cannot place an element above content outside its ancestor's stacking context.

## Alternatives & Trade-offs

Flexbox and Grid distribute ordinary layout space more predictably than offsets and absolute positioning. Positioning is precise for transient overlays but requires care for collision, scrolling, zoom, and focus behaviour.

## Common Mistakes

- Solving an ordinary content layout with absolute positioning or arbitrary offsets.
- Forgetting that absolutely positioned content no longer reserves space.
- Assuming `z-index: 9999` always places an element above everything else.
- Using `sticky` without accounting for its scroll container or inset threshold.

## Common Interview Questions

### Foundation

- What is normal flow?
- How do `relative`, `absolute`, `fixed`, and `sticky` positioning differ?
- What is a containing block?

### Intermediate

- How would you anchor a badge to a card without using it for the card's layout?
- Why might a fixed element behave unexpectedly inside a transformed ancestor?
- Why does an element with a high `z-index` appear behind another element?

### Advanced and Follow-up

- How would you debug an overlay that is clipped or layered beneath a modal?
- What accessibility concerns apply when an overlay is opened?

## Practical Tasks

- Anchor a status badge to a card while keeping the card content in normal flow.
- Explain a stacking-context bug using the element tree and computed styles.
- Replace an absolutely positioned content layout with flow, Flexbox, or Grid.

## Readiness Criteria

You can preserve normal flow for ordinary content, choose an appropriate positioning mode for overlays, and diagnose containing-block and stacking-context issues.

## References

- [MDN: Positioning](https://developer.mozilla.org/docs/Learn/CSS/CSS_layout/Positioning)
- [MDN: Stacking context](https://developer.mozilla.org/docs/Web/CSS/CSS_positioned_layout/Stacking_context)