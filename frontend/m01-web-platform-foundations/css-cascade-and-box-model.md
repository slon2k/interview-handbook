# CSS Cascade and Box Model

## Definition

CSS declarations are resolved by the cascade, which considers origin, importance, layers, specificity, and source order. The box model determines the space an element occupies through its content, padding, border, and margin.

## How It Works

- Use the browser's computed-style and box-model panels to find which declaration wins and where space is being added.
- Understand specificity and prefer low-specificity selectors that remain easy to override.
- Inherited properties come from ancestors; non-inherited properties use their initial or specified values.
- Custom properties provide runtime design tokens and component-level variation, and they participate in inheritance.
- `box-sizing: border-box` makes declared width and height include padding and border, which often makes layout sizing easier to reason about.
- Positioning depends on the containing block. Stacking contexts isolate `z-index` comparisons, so a large value cannot escape its context.

## Alternatives & Trade-offs

Utility classes make local styles explicit and quick to apply. CSS Modules or similar scoping reduces naming collisions. Shared styles centralise tokens and global rules but need layering and naming discipline. The best choice depends on the team, tooling, reuse, and debugging cost.

## Common Mistakes

- Solving every conflict with `!important` or a larger selector.
- Forgetting that padding and borders affect an element's rendered size.
- Assuming `z-index: 9999` always places an element above everything else.
- Hiding overflow to mask a sizing problem, clipping content or focus.
- Defining custom properties globally without considering inheritance and component boundaries.

## Common Interview Questions

### Foundation

- How does the box model work?
- How are conflicting CSS declarations resolved?
- What is the difference between inheritance and the cascade?

### Intermediate

- Why might a width of `100%` still overflow its container?
- Why does a child with a high `z-index` appear behind another element?
- When would you use a custom property?

### Advanced and Follow-up

- How would you reduce a specificity conflict in a growing React application?
- What browser tools would you use to diagnose unexpected spacing or a stacking bug?

## Practical Tasks

- Diagnose a component whose declared width overflows after padding is added.
- Reduce a set of competing selectors to a predictable cascade without adding `!important`.
- Explain a stacking-context bug using the element tree and computed styles.

## Readiness Criteria

You can predict common cascade and box-model outcomes, use computed styles to verify a hypothesis, and justify a maintainable CSS organisation strategy.

## References

- [MDN: Introduction to the CSS box model](https://developer.mozilla.org/docs/Web/CSS/CSS_box_model/Introduction_to_the_CSS_box_model)
- [MDN: The cascade](https://developer.mozilla.org/docs/Web/CSS/CSS_cascade/Cascade)
- [MDN: Stacking context](https://developer.mozilla.org/docs/Web/CSS/CSS_positioned_layout/Stacking_context)
