# Forms, Validation, and Submission

## Definition

A React form collects user input, displays validation feedback, and submits a deliberate payload to an API. Controlled and uncontrolled forms represent different ownership choices for input values, while validation can happen locally, at the server boundary, or both.

## How It Works

- A controlled input receives its value from React state and reports changes through an event handler.
- An uncontrolled input keeps its current value in the DOM and can be read through a ref or form submission APIs.
- Client validation can provide immediate feedback, but server validation remains authoritative for business rules.
- A submission should model idle, editing, submitting, success, and error states without allowing duplicate or invalid requests.
- Server field errors should map back to stable field names and remain visible without losing a general error message.

## Application

Use controlled inputs when UI behavior depends on each change. Use uncontrolled inputs or a form library when the form is large and field-level rerendering or registration concerns matter. Keep the final submission contract aligned with the typed API DTO rather than sending display state directly.

### Typed submission and server failures

Keep a form draft separate from the API payload when display values, optionality, or normalization differ. Model submission as explicit states so a failure preserves the user's draft and a retry has a clear transition. The framework-independent state shapes belong in [Module 3](../m03-typescript/typing-ui-state-forms-and-async-results.md); React event types and component contracts are covered in [TypeScript components, props, events, and generics](typescript-components-props-events-and-generics.md).

```tsx
type SaveState =
  | { status: "editing" }
  | { status: "submitting" }
  | { status: "validation-error"; fieldErrors: Record<string, string[]> }
  | { status: "conflict"; message: string }
  | { status: "success" };
```

Map ASP.NET Core `ValidationProblemDetails.errors` to stable form-field names, preserving every message for a field. Do not treat `401` or `403` as validation failures: those belong to the auth flow. A `409` conflict needs a deliberate recovery path such as reload, compare, or retry; an unexpected `5xx` needs a general recoverable error. See [Module 4: API contracts](../m04-browser-platform-and-aspnet-core-api-integration/api-dtos-pagination-and-validation-errors.md) and [auth status handling](../m04-browser-platform-and-aspnet-core-api-integration/cookies-storage-auth-and-status-handling.md).

## Common Mistakes

- Treating client validation as a substitute for server validation.
- Updating a controlled input from a stale value or switching between controlled and uncontrolled modes.
- Disabling the submit button without representing server failure or retry behavior.
- Clearing field errors on every keystroke when the user still needs the feedback.
- Sending formatted display values where the API expects a normalized DTO.
- Replacing a field-level validation response with one generic message and losing actionable feedback.
- Treating `401`, `403`, or `409` as though they were retryable field-validation failures.

## Common Interview Questions

### Foundation

- What is the difference between controlled and uncontrolled inputs?
- Why does a form need server-side validation even when it validates in the browser?

### Intermediate

- How would you represent submitting and validation-error states?
- When might a form library be useful?
- How would you type a submit event and keep a draft separate from the API payload?

### Advanced and Follow-up

- How do you prevent duplicate submissions while still allowing a retry after failure?
- How would you map an ASP.NET Core validation error dictionary to fields in a React form?
- How should `400` validation, `401`, `403`, `409`, and `500` produce different user-facing behavior?

### Code Prediction

Given a controlled input whose `value` becomes `undefined` after a failed load, explain the controlled/uncontrolled warning and how to model the empty value safely.

## Practical Tasks

- Build a typed controlled form with local validation and server field-error mapping.
- Compare a controlled form with an uncontrolled form and justify the choice for a large data-entry workflow.
- Preserve a user's draft after validation failure and add separate recovery behavior for conflict and authorization responses.

## Readiness Criteria

You can choose controlled or uncontrolled inputs, model submission states, preserve server validation details, and keep form data separate from transport and display models.

## References

- [React: Sharing state between components](https://react.dev/learn/sharing-state-between-components)
- [React: `useState`](https://react.dev/reference/react/useState)
- [React Hook Form documentation](https://react-hook-form.com/)
