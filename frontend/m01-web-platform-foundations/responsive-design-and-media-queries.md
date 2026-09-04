# Responsive Design and Media Queries

## Definition

Responsive design adapts one interface to the space, media, input method, and user preferences available at runtime. Media and container queries let CSS respond to those conditions without creating separate device-specific implementations.

## How It Works

- Start with content in normal flow and add layout constraints that work at the smallest practical width.
- Let breakpoints follow content failure, such as controls becoming unreadable or a column too narrow, rather than a named device list.
- Use media queries for viewport- and environment-level conditions, such as width, pointer capability, colour scheme, and `prefers-reduced-motion`.
- Use container queries when a reusable component must adapt to its own available inline size rather than the viewport.
- Use relative units such as `rem`, `%`, `ch`, and viewport or container units when they express the real constraint; pixels remain useful where a fixed physical relationship is intended.
- Give images intrinsic dimensions where possible, constrain them with `max-width: 100%`, and select `object-fit` according to whether cropping is acceptable.

## Alternatives & Trade-offs

Viewport media queries are direct and appropriate for page-level changes. Container queries make components more portable but require a deliberate containment boundary. A few meaningful adaptive changes are easier to maintain and test than many narrow device overrides.

## Common Mistakes

- Building a desktop-only layout and patching it later with many media queries.
- Choosing breakpoints by phone or tablet name instead of content constraints.
- Using fixed widths that clip content at zoom or narrow widths.
- Ignoring reduced-motion preferences for non-essential motion.
- Distorting images by applying both dimensions without considering aspect ratio or `object-fit`.

## Common Interview Questions

### Foundation

- What makes a layout responsive?
- How do media queries and container queries differ?
- How would you make an image responsive?

### Intermediate

- How would you build a two-column layout that becomes one column without duplicating markup?
- How do you choose a breakpoint?
- When is a container query more appropriate than a media query?

### Advanced and Follow-up

- How would you prevent layout shift while images or asynchronous content load?
- How should an interface respond to `prefers-reduced-motion`?
- How would you test a responsive component that is reused in a narrow sidebar and a wide page?

## Practical Tasks

- Build a two-column layout that becomes one column based on available space.
- Add a container query to a reusable card component with compact and expanded presentations.
- Add responsive image constraints and reduced-motion handling without changing a component's content.

## Readiness Criteria

You can build and test an interface at narrow and wide sizes, choose queries based on the boundary that owns the constraint, and explain responsive trade-offs.

## References

- [MDN: Responsive design](https://developer.mozilla.org/docs/Learn/CSS/CSS_layout/Responsive_Design)
- [MDN: Using media queries](https://developer.mozilla.org/docs/Web/CSS/CSS_media_queries/Using_media_queries)
- [MDN: CSS container queries](https://developer.mozilla.org/docs/Web/CSS/CSS_containment/Container_queries)