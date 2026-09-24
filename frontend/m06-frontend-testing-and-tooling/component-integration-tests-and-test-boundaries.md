# Component Integration Tests and Test Boundaries

## Definition

A component integration test renders a feature with the providers, router state, and API boundary it needs, then verifies a user-visible result. It is broader than a pure function or hook test but narrower and faster than an end-to-end browser journey.

## Alternatives & Trade-offs

Hook-only tests can be useful for reusable stateful logic with a meaningful public API. They cannot prove that a component labels a control, wires an event, displays an error, or integrates correctly with a route. Rendering the feature through its public component boundary gives stronger confidence but requires deliberate test helpers.

## How It Works

Create a `renderWithProviders` helper only for stable, app-level dependencies such as theme, router, or query client. Let each test declare meaningful route state and request scenarios explicitly; hidden defaults make a test harder to read and can conceal a dependency it should own.

```tsx
renderWithProviders(<ProductPage />, { route: "/products?query=keyboard" });
expect(await screen.findByRole("heading", { name: "Products" })).toBeVisible();
```

## Application

Test a routed page, form, or data-driven feature through the controls and UI states users encounter. Use a hook test for logic whose callers form the correct observable boundary, such as a reusable debouncing hook, then separately test the component that consumes it.

## Common Mistakes

- Rendering a component without required providers and then mocking internal imports to make it pass.
- Putting all providers and all application state in every test helper.
- Testing an implementation hook while never testing the rendered screen.
- Letting a helper choose an unmentioned route or authenticated user.

## Common Interview Questions

### Foundation

- What distinguishes a component integration test from an isolated hook test?

### Intermediate

- Which providers belong in a shared render helper?
- How would you test a route parameter changes the displayed data?

### Advanced and Follow-up

- How can a broad test helper make tests less trustworthy?

### Code Prediction

Why might a hook test pass while the actual form using that hook remains inaccessible to keyboard users?

## Practical Tasks

- Build a focused render helper for a routed feature and document its explicit inputs.
- Test a route-param change, an MSW response, and the corresponding rendered state.

## Readiness Criteria

You can select a component or hook boundary intentionally, assemble only needed collaborators, and prove a feature works through its user-facing interface.

## References

- [Testing Library: setup](https://testing-library.com/docs/react-testing-library/setup/)
- [React Router testing](https://reactrouter.com/start/framework/testing)
