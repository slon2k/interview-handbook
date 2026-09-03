# Accessibility and Keyboard Interaction

## Definition

Accessible interfaces can be perceived, operated, and understood by people with different abilities and assistive technologies. Keyboard access, focus management, accessible names, contrast, and appropriate semantics are core implementation concerns rather than optional polish.

## How It Works

- Keep focus visible and in a logical order. Do not remove an element from the focus order without a reason.
- An accessible name usually comes from visible text, an associated `label`, or a deliberate naming attribute. Placeholder text is not a label.
- A dialog needs focus placement on entry, appropriate keyboard handling while open, and focus restoration when closed.
- Use ARIA to supplement native HTML when necessary. Incorrect or redundant ARIA can contradict the browser's native meaning.
- Do not communicate status, errors, or required fields through colour alone. Check text and control contrast and support zoom and text resizing.
- Test with keyboard navigation, the browser accessibility tree, automated checks, and representative assistive technology workflows.

## Alternatives & Trade-offs

Automated tools are fast and useful for repeatable checks, but they cannot establish that an interaction is understandable or that focus moves correctly. Manual keyboard testing catches workflow problems; screen-reader testing adds another perspective but should not be treated as a substitute for semantic implementation.

## Common Mistakes

- Making a custom control clickable but not keyboard-operable.
- Moving focus unexpectedly during ordinary navigation.
- Hiding the focused element or trapping focus without an escape path.
- Adding `tabindex="1"` or a large number of positive tabindex values.
- Treating an accessibility scan with no reported violations as proof that the UI is accessible.

## Common Interview Questions

### Foundation

- What is an accessible name?
- How do you test a page without using a mouse?
- When should you use ARIA?

### Intermediate

- How would you make a modal dialog accessible?
- How should a form error be announced and associated with its input?
- What is wrong with using `tabindex="1"` to control focus order?

### Advanced and Follow-up

- What would you investigate if automated checks pass but keyboard users report that a workflow is confusing?
- How would you design an autocomplete so keyboard users can inspect and select suggestions?

## Practical Tasks

- Review a dialog for focus entry, focus containment, Escape handling, and focus restoration.
- Navigate a page using only the keyboard and record every point where focus is lost or the next action is unclear.

## Readiness Criteria

You can explain accessible names and focus, identify inappropriate ARIA, design keyboard behaviour for common interactions, and describe a layered accessibility-testing approach.

## References

- [MDN: Accessibility](https://developer.mozilla.org/docs/Web/Accessibility)
- [WAI-ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)
- [W3C: Web Content Accessibility Guidelines](https://www.w3.org/TR/WCAG22/)
