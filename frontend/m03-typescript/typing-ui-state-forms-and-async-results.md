# Typing UI State, Forms, Callbacks, and Async Results

## Definition

UI types should represent the states and events a feature can actually have: empty or populated values, editing or submitting forms, and pending, successful, empty, or failed asynchronous work. A good model makes invalid combinations difficult to express.

## How It Works

- Use literal unions or discriminated unions for finite UI states instead of unrelated booleans that can conflict.
- Keep event and callback types at the boundary where they are consumed; framework-specific event types belong in the React module.
- Separate a form's draft model from a server DTO when field names, optionality, or validation timing differ.
- Represent async results with a discriminant and state-specific fields rather than `data?: T`, `loading: boolean`, and `error?: string` combinations.
- Generic result types such as `AsyncState<T>` preserve the data type across loading and success branches.

## Application

Type the state transitions of a form or data view before implementing handlers. This exposes missing transitions, prevents success-only assumptions, and creates a stable contract for Module 5 React components.

This lesson supplies the type model. [Module 4's fetch lesson](../m04-browser-platform-and-aspnet-core-api-integration/fetch-requests-cancellation-and-stale-responses.md) covers request cancellation and stale-response control, while [Module 5's async UI lesson](../m05-react/typed-data-fetching-and-async-ui-states.md) applies these states during React rendering.

## Common Mistakes

- Combining booleans that permit impossible states such as loading and success with no data.
- Reusing a server DTO directly for every form state.
- Using `Function` or `any` for callbacks.
- Making all fields optional to avoid modeling the empty state.

## Common Interview Questions

### Foundation

- Why are discriminated unions useful for UI state?
- How should a form model differ from an API DTO?

### Intermediate

- How would you type loading, success, empty, and error states for a list?
- What belongs in a callback's function type?

### Advanced and Follow-up

- How would you prevent a submit handler from receiving a draft that is not valid for the API?

### Code Prediction

Given an `AsyncState<User[]>` union, identify which properties are available in the `status: "success"` branch and why `data` is not available in the loading branch.

## Practical Tasks

- Replace a collection of UI booleans with a discriminated async-state union.
- Model a form's draft, validation error, submitting, and success states without non-null assertions.

## Readiness Criteria

You can model UI state transitions honestly and carry precise data types through callbacks and async branches.

## References

- [TypeScript Handbook: Discriminated unions](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions)
- [TypeScript Handbook: Function type expressions](https://www.typescriptlang.org/docs/handbook/2/functions.html#function-type-expressions)