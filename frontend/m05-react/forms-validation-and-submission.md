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

## Common Mistakes

- Treating client validation as a substitute for server validation.
- Updating a controlled input from a stale value or switching between controlled and uncontrolled modes.
- Disabling the submit button without representing server failure or retry behavior.
- Clearing field errors on every keystroke when the user still needs the feedback.
- Sending formatted display values where the API expects a normalized DTO.

## Common Interview Questions

### Foundation

- What is the difference between controlled and uncontrolled inputs?
- Why does a form need server-side validation even when it validates in the browser?

### Intermediate

- How would you represent submitting and validation-error states?
- When might a form library be useful?

### Advanced and Follow-up

- How do you prevent duplicate submissions while still allowing a retry after failure?
- How would you map an ASP.NET Core validation error dictionary to fields in a React form?

### Code Prediction

Given a controlled input whose `value` becomes `undefined` after a failed load, explain the controlled/uncontrolled warning and how to model the empty value safely.

## Practical Tasks

- Build a typed controlled form with local validation and server field-error mapping.
- Compare a controlled form with an uncontrolled form and justify the choice for a large data-entry workflow.

## Readiness Criteria

You can choose controlled or uncontrolled inputs, model submission states, preserve server validation details, and keep form data separate from transport and display models.

## References

- [React: Sharing state between components](https://react.dev/learn/sharing-state-between-components)
- [React: `useState`](https://react.dev/reference/react/useState)
- [React Hook Form documentation](https://react-hook-form.com/)
