# Responsive Design and Media Queries

## Definition

Responsive design means a layout adapts to the actual space, device, and user preferences available, rather than being built for one fixed viewport size. Media queries apply CSS conditionally based on viewport width (most commonly), device characteristics, or user preferences like reduced motion or color scheme — but modern responsive CSS increasingly reaches for intrinsically flexible techniques (Grid's `auto-fit`, `clamp()`) before falling back to explicit breakpoints at all.

```css
.card { padding: 16px; }

@media (min-width: 768px) {
  .card { padding: 24px; }
}
```

## Alternatives & Trade-offs

Media-query breakpoints give precise, deliberate control over how a layout changes at specific viewport widths, but require choosing those exact breakpoints and maintaining them as content and devices change — a layout can look correct at the tested breakpoints and break at a width in between. Intrinsically responsive techniques (Grid's `auto-fit`/`minmax`, `clamp()` for fluid typography) adapt continuously with no fixed breakpoints to maintain, at the cost of somewhat less precise control over the exact appearance at any given width.

## How It Works

### Mobile-first — the default, deliberate ordering of media queries

```css
/* Base styles target the SMALLEST/simplest case first */
.nav { display: block; }

/* min-width media queries progressively ADD complexity for larger viewports */
@media (min-width: 768px) {
  .nav { display: flex; }
}
```

Writing base styles for mobile first and layering `min-width` media queries on top (rather than `max-width` queries narrowing down from a desktop-first base) tends to produce simpler CSS overall, since most content naturally needs to gain complexity (more columns, more visible chrome) as space increases, rather than needing to be stripped down.

### `rem`/`em` vs. `px` — respecting user font-size preferences

```css
html { font-size: 16px; } /* the root font size, which rem is relative to */
.heading { font-size: 2rem; } /* 32px, but SCALES if the user's browser default font size changes */
.heading-fixed { font-size: 32px; } /* stays exactly 32px regardless of user preference */
```

A user who has increased their browser's default font size for readability expects text sized in `rem` to scale accordingly — text hardcoded in `px` ignores that preference entirely, a real accessibility consideration, not just a stylistic one.

### `clamp()` for fluid values with no breakpoint at all

```css
.heading {
  font-size: clamp(1.5rem, 4vw + 1rem, 3rem);
  /* never smaller than 1.5rem, never larger than 3rem, scales fluidly with viewport width in between */
}
```

`clamp(minimum, preferred, maximum)` lets a value scale continuously with viewport size while still being bounded on both ends — avoiding both an explicit media query and the risk of text becoming unreadably small or absurdly large at extreme viewport widths.

### Viewport meta tag — required for mobile browsers to render responsive CSS at all

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

Without this tag, mobile browsers render the page at a fixed desktop-width viewport (typically 980px) and then zoom it out to fit the screen — media queries technically apply, but based on that fake desktop-width viewport, not the phone's actual physical width, making the whole responsive design invisible until this tag is added.

### `prefers-reduced-motion` and `prefers-color-scheme` — responding to user preferences, not just device size

```css
@media (prefers-reduced-motion: reduce) {
  * { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }
}

@media (prefers-color-scheme: dark) {
  body { background: #111; color: #eee; }
}
```

"Responsive" isn't only about screen size — respecting a user's OS-level motion-sensitivity or dark-mode preference is the same underlying idea (adapting to the user's actual context) applied to a different signal than viewport width.

## Application

Write mobile-first CSS, using `min-width` media queries to progressively add complexity for larger viewports. Use `rem` for font sizes and most spacing to respect user font-size preferences. Reach for `clamp()` for fluid values before adding an explicit breakpoint. Always include the viewport meta tag. Respect `prefers-reduced-motion` and `prefers-color-scheme` where relevant.

## Common Mistakes

- Omitting the viewport meta tag, causing mobile browsers to render at a fixed desktop-width viewport regardless of any media queries written.
- Writing desktop-first CSS with `max-width` queries stripping features away, instead of mobile-first `min-width` queries adding them.
- Hardcoding font sizes in `px`, ignoring users who have changed their browser's default font size for readability.
- Choosing breakpoints based on specific device widths rather than where the actual content or layout starts to break, missing devices that don't match the assumed sizes.
- Ignoring `prefers-reduced-motion`, causing animations to actively harm users with vestibular disorders rather than just being a stylistic choice.

## Common Interview Questions

### Basic
- What does "mobile-first" mean in the context of media queries?
- Why is the viewport meta tag necessary for responsive design to work on mobile browsers?

### Intermediate
- Why might `rem` be preferred over `px` for font sizes?
- What does `clamp()` do, and when would you reach for it instead of a media query?

### Advanced
- How would you decide where to place a breakpoint, rather than picking a specific device width?
- Why does respecting `prefers-reduced-motion` matter beyond aesthetic preference?

### Follow-up Questions
- Does omitting the viewport meta tag cause media queries to stop working entirely, or just to apply against the wrong viewport width?
- Can `clamp()` fully replace the need for any media queries in a design?

### Code Prediction
Given a page missing the viewport meta tag, with a media query `@media (max-width: 480px)`, would this media query ever apply on an actual mobile phone with a 390px-wide screen? Why or why not?

## Practical Tasks

- Add the viewport meta tag to a page missing it and observe the difference in how media queries behave on a simulated mobile width.
- Convert a desktop-first stylesheet using `max-width` queries into a mobile-first one using `min-width` queries.
- Replace a font-size breakpoint with a `clamp()`-based fluid value and compare the result across viewport widths.

## Readiness Criteria

Write mobile-first responsive CSS, explain why the viewport meta tag is required, use `clamp()` for fluid values where appropriate, and account for user preferences like reduced motion alongside viewport-based responsiveness.

## References

- [MDN: Responsive design](https://developer.mozilla.org/docs/Learn/CSS/CSS_layout/Responsive_Design)
- [MDN: Using media queries](https://developer.mozilla.org/docs/Web/CSS/CSS_media_queries/Using_media_queries)
- [MDN: prefers-reduced-motion](https://developer.mozilla.org/docs/Web/CSS/@media/prefers-reduced-motion)
