# React Testing Library Queries and User Events

## Definition

React Testing Library (RTL) tests a rendered component through the semantics and interactions a user can observe. Prefer queries by role, accessible name, label, text, and value over component internals, CSS selectors, or implementation-specific test IDs.

## Alternatives & Trade-offs

`getByRole` makes accessibility semantics part of the test contract and fails with useful output when the UI is not discoverable. A `data-testid` is reasonable when no accessible query distinguishes repeated or non-semantic content, but using it by default can allow inaccessible UI to pass tests.

## How It Works

Render the component, locate an accessible control, and simulate the interaction through `userEvent`:

```tsx
const user = userEvent.setup();
render(<SearchForm onSearch={onSearch} />);

await user.type(screen.getByLabelText("Search products"), "keyboard");
await user.click(screen.getByRole("button", { name: "Search" }));

expect(onSearch).toHaveBeenCalledWith("keyboard");
```

`userEvent` models a sequence of user interactions more faithfully than directly invoking props or dispatching a low-level event. Tests should assert changed visible state, navigation, or collaborator effects that matter to the feature.

## Application

Test buttons by role and name, fields by label, status messages by role or visible text, and dialogs by their accessible semantics. Arrange a component through public props and providers, perform an interaction, then assert its observable result.

## Common Mistakes

- Selecting nodes with class names or component-library DOM structure.
- Calling internal callback props instead of using the control a user operates.
- Asserting every implementation detail of a component rather than its outcome.
- Defaulting to `data-testid` when a missing role or label is the actual defect.

## Common Interview Questions

### Foundation

- Why does RTL prefer accessible queries?
- When is `data-testid` justified?

### Intermediate

- Why use `userEvent` instead of directly calling an event handler?
- How does a role-based test improve accessibility confidence?

### Advanced and Follow-up

- How would you test two visually identical controls with different user purposes?

### Code Prediction

What should a role-based query do when an icon-only button has no accessible name, and why is that helpful?

## Practical Tasks

- Replace CSS-selector queries in an existing test with role, label, and name queries.
- Test a keyboard interaction using `userEvent` and assert the resulting visible state.

## Readiness Criteria

You can select elements through accessible semantics, model a user interaction, and distinguish a useful test seam from an implementation detail.

## References

- [Testing Library queries](https://testing-library.com/docs/queries/about/)
- [Testing Library user-event](https://testing-library.com/docs/user-event/intro/)
