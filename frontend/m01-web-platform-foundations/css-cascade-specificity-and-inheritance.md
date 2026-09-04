# CSS Cascade, Specificity, and Inheritance

## Definition

The cascade is the process the browser uses to select a value when multiple CSS declarations apply. It considers origin, importance, cascade layers, specificity, and source order. Inheritance is a separate process through which some property values pass from an ancestor to its descendants.

## How It Works

- Inspect the browser's computed styles to see the winning declaration and every overridden alternative.
- Prefer selectors with low, predictable specificity so component styles remain easy to override.
- Understand that author `!important` declarations should be exceptional; increasing specificity to win a conflict usually increases future maintenance cost.
- Know which properties inherit, such as `color` and `font-family`, and which normally do not, such as `margin` and `border`.
- Use custom properties for design tokens and deliberate variation. They inherit by default, so define them at the narrowest useful boundary.
- Use cascade layers when a shared stylesheet needs an explicit, documented ordering between reset, base, component, and utility rules.

## Alternatives & Trade-offs

CSS Modules reduce accidental cross-component conflicts through scoping. Utility classes make local styling explicit and avoid selector naming, while shared global CSS can centralise tokens and baseline rules. The right choice depends on the team's conventions, reuse needs, and debugging cost; none removes the need to understand the cascade.

## Common Mistakes

- Solving every conflict with `!important` or a more specific selector.
- Confusing inheritance with the cascade.
- Defining a custom property globally when it should vary by component or theme boundary.
- Depending on source order that is accidental or obscured by a build tool.

## Common Interview Questions

### Foundation

- How does the browser decide which conflicting CSS declaration wins?
- What is specificity, and how does it differ from inheritance?
- Which kinds of properties commonly inherit?

### Intermediate

- How would you diagnose why a component rule is being overridden?
- When would a custom property be preferable to repeating a literal value?
- Why can a more specific selector be a maintenance problem?

### Advanced and Follow-up

- How would you reduce specificity conflicts in a growing React application?
- What role do cascade layers play in a stylesheet containing third-party CSS and utilities?

## Practical Tasks

- Reduce a set of competing selectors to a predictable cascade without adding `!important`.
- Use browser developer tools to identify the declaration that wins for a broken component style.
- Introduce component-scoped custom properties for a visual variant without duplicating a rule.

## Readiness Criteria

You can predict common cascade and inheritance outcomes, inspect computed styles to verify a hypothesis, and justify a CSS organisation strategy that stays overrideable.

## References

- [MDN: The cascade](https://developer.mozilla.org/docs/Web/CSS/CSS_cascade/Cascade)
- [MDN: Specificity](https://developer.mozilla.org/docs/Web/CSS/CSS_cascade/Specificity)
- [MDN: Inheritance](https://developer.mozilla.org/docs/Web/CSS/CSS_cascade/Inheritance)