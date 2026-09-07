# Accessibility and Keyboard Interaction

## Definition

Accessibility means the interface works for people using a keyboard alone, a screen reader, or with low vision — not an add-on feature, but a property of correctly-built markup (previous two topics) plus deliberate attention to focus order, accessible naming, and color contrast. ARIA (Accessible Rich Internet Applications) attributes exist specifically to describe custom widgets that have no native HTML equivalent — they never change behavior, only how assistive technology describes what's already there.

```html
<!-- Native control: accessible name, keyboard behavior, and role all come from the element itself -->
<button>Close</button>

<!-- Custom widget with no native equivalent: ARIA describes what it IS, but doesn't make it behave that way -->
<div role="button" tabindex="0" aria-label="Close">×</div>
```

## Alternatives & Trade-offs

Relying entirely on native elements (previous two topics) is simpler and gets keyboard/screen-reader behavior for free, but some UI patterns (a tab panel, a custom combobox, a modal dialog) genuinely have no native HTML equivalent. For those, ARIA lets you describe the correct role and state, but you must still implement the actual keyboard behavior yourself — ARIA attributes alone add no interaction, and adding a `role` without the matching keyboard behavior is often worse than no ARIA at all, since it promises behavior to assistive technology that doesn't actually exist.

## How It Works

### The first rule of ARIA: don't use ARIA if a native element already does the job

```html
<!-- Unnecessary and redundant: <button> already has role="button" built in -->
<button role="button">Submit</button>

<!-- Actively harmful: overrides a native element's own correct semantics -->
<h2 role="presentation">Section Title</h2>
```

Adding ARIA to a native element that already has the correct role is redundant at best; overriding a native element's role can actively strip away correct behavior that was already there for free.

### The accessible name — what a screen reader actually announces

```html
<!-- No accessible name at all: a screen reader announces "button", nothing more -->
<button><svg>...</svg></button>

<!-- Accessible name provided explicitly, since there's no visible text for the browser to use -->
<button aria-label="Close dialog"><svg>...</svg></button>

<!-- Alternative: visually hidden text serves the same purpose without relying on aria-label -->
<button><span class="visually-hidden">Close dialog</span><svg>...</svg></button>
```

An icon-only button with no visible text and no `aria-label` has no accessible name at all — a screen reader user hears only "button," with no indication of what it does.

### Focus order should match visual and logical reading order

```html
<!-- WRONG: tabindex values force an unintuitive tab order, unrelated to visual layout -->
<input tabindex="3">
<input tabindex="1">
<input tabindex="2">

<!-- RIGHT: rely on natural DOM order for focus order; avoid positive tabindex values entirely -->
<input>
<input>
<input>
```

Positive `tabindex` values are almost always a mistake — they create a custom tab order that's easy to get subtly wrong and hard to maintain as the page changes. `tabindex="0"` (making a non-interactive element focusable in its natural DOM position) and `tabindex="-1"` (removing an element from the tab order while still allowing it to receive focus programmatically) are the two values actually worth using.

### Managing focus for dynamic content — a modal dialog example

```javascript
function openModal(modalElement, triggerButton) {
  modalElement.showModal();          // or toggling visibility manually
  modalElement.querySelector('button, input, [tabindex]')?.focus(); // move focus INTO the modal
}

function closeModal(modalElement, triggerButton) {
  modalElement.close();
  triggerButton.focus();              // return focus to where the user was, not to <body>
}
```

Without explicit focus management, opening a modal can leave keyboard focus behind the modal on a now-hidden or now-obscured element — the user has no idea where they are. Closing it without restoring focus to the trigger loses their place in the page entirely.

### Color contrast — a testable, objective requirement

```
WCAG AA requires a contrast ratio of at least 4.5:1 for normal text, 3:1 for large text
(≥18pt, or ≥14pt bold) — this is measured objectively with a contrast-checking tool, not
judged by eye, since what looks "readable enough" varies wildly by individual vision.
```

## Application

Prefer native elements whenever one exists for the pattern you need (previous two topics), and reach for ARIA only to describe genuinely custom widgets with no native equivalent — pairing every ARIA role with the matching keyboard behavior, never one without the other. Manage focus explicitly for anything that opens, closes, or replaces content dynamically. Check color contrast with a tool, not by eye.

## Common Mistakes

- Adding an ARIA role to a native element that already has that role built in, or overriding a native element's correct semantics.
- Giving an icon-only button no accessible name at all, leaving screen reader users with no indication of its purpose.
- Using positive `tabindex` values to force a custom tab order, creating a fragile, hard-to-maintain focus sequence.
- Opening a modal or replacing page content without moving focus into it, and without restoring focus when it closes.
- Judging color contrast by eye instead of measuring it against the WCAG AA thresholds with a tool.
- Adding a `role` to describe a widget's behavior without actually implementing the keyboard interaction that role implies.

## Common Interview Questions

### Basic
- What is the "first rule of ARIA"?
- What does an accessible name mean, and how would you provide one for an icon-only button?

### Intermediate
- Why are positive `tabindex` values generally considered a mistake?
- Why must focus be moved explicitly when a modal opens, rather than relying on default browser behavior?

### Advanced
- What's the risk of adding an ARIA role to a custom widget without also implementing the corresponding keyboard behavior?
- How would you verify a page's color contrast meets WCAG AA objectively, rather than relying on visual judgment?

### Follow-up Questions
- Does `tabindex="-1"` remove an element from the tab order entirely, or just from the sequential Tab-key order?
- Is ARIA ever necessary for a page built entirely from native HTML elements used correctly?

### Code Prediction
Given `<button><svg>...</svg></button>` with no `aria-label` and no visible text content, what does a screen reader announce when a user tabs to this button? What's the minimal change that would fix it?

## Practical Tasks

- Add accessible names to a set of icon-only buttons using `aria-label` or visually-hidden text.
- Implement focus management for a modal dialog: moving focus in on open, and restoring it to the trigger on close.
- Audit a page's text/background color combinations against WCAG AA contrast thresholds using a contrast-checking tool.
- Identify and remove unnecessary positive `tabindex` values from a sample page, relying on natural DOM order instead.

## Readiness Criteria

Apply the "don't override native semantics" rule, provide accessible names for icon-only controls, manage focus explicitly for dynamic content, and verify color contrast objectively rather than by eye.

## References

- [MDN: ARIA](https://developer.mozilla.org/docs/Web/Accessibility/ARIA)
- [MDN: Keyboard-navigable JavaScript widgets](https://developer.mozilla.org/docs/Web/Accessibility/Guides/Keyboard-navigable_JavaScript_widgets)
- [WCAG 2.1 Understanding Contrast (Minimum)](https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum.html)
