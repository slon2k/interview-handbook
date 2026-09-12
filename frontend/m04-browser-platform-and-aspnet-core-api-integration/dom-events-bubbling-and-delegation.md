# DOM Events, Bubbling, Capturing, and Delegation

## Definition

DOM events are notifications raised by the browser when the user interacts with the page, the document changes, or browser APIs emit activity. Bubbling and capturing describe the path that an event takes through the DOM tree, and delegation is a pattern where a parent handler listens for events from many child elements.

## How It Works

- The browser creates an event when a user clicks, types, submits a form, or a browser API emits activity.
- Capturing runs from the document root down to the target node; bubbling runs from the target back up to the root.
- `event.target` identifies the actual element that caused the event; `event.currentTarget` identifies the element currently handling it.
- Event delegation works because the event bubbles to a common ancestor so one listener can handle many items.
- `preventDefault()` stops default browser behavior; `stopPropagation()` prevents further propagation.

## Application

Use event delegation for lists, tables, and repeated UI items where the number of descendants can change. It reduces listeners and keeps behavior consistent as the DOM updates.

## Common Mistakes

- Capturing or bubbling incorrectly but assuming the DOM is static.
- Using `stopPropagation()` without understanding why the parent or sibling handlers should still run.
- Attaching a click handler to each item in a list instead of delegating to the parent container.
- Reading `event.target` and assuming it is the element with the listener attached.

## Common Interview Questions

### Foundation

- What is the difference between `event.target` and `event.currentTarget`?
- What does bubbling mean?

### Intermediate

- Why is event delegation useful for a dynamic list?
- How does capturing differ from bubbling?

### Advanced and Follow-up

- What happens if a child handler calls `stopPropagation()` and a parent also needs to react?
- Why might a delegated click handler need to inspect `closest()` or `dataset`?

### Code Prediction

Given a button inside a list item inside a container, predict which handler runs first when a click occurs and what value you would inspect to know which actual item was clicked.

## Practical Tasks

- Refactor a static list of buttons to use one delegated listener.
- Explain how a form submission event propagates and how a submit handler can inspect the submitted field values.

## Readiness Criteria

You can reason about bubbling, capturing, and delegation and explain how to choose the correct target and listener location for interactive UI code.

## References

- [MDN: Event bubbling](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events)
- [MDN: EventTarget.addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)
- [MDN: `closest()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/closest)
