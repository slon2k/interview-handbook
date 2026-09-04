# Forms and Native Controls

## Definition

A form collects user input and submits a meaningful set of named controls. Native controls such as text inputs, checkboxes, radio buttons, selects, and buttons provide browser behaviour that custom controls must otherwise recreate.

## Application

Start with a native `<form>` and native controls, then layer client feedback and API error mapping on top. Keep browser validation as fast user feedback only: an ASP.NET Core endpoint must independently validate every submitted value and return errors the UI can associate with the relevant fields.

## How It Works

- Associate every control with a visible `label`; use `fieldset` and `legend` for related groups.
- Choose input types and constraints that match the data, such as `email`, `number`, `required`, `min`, and `pattern`.
- Give controls useful names so submitted data and automated tests can identify them.
- Present instructions and validation errors in text, close to the relevant control, without relying on colour alone.
- Preserve entered values when validation fails and identify which fields need attention.
- Client-side validation improves feedback speed but is not a security boundary. Server-side validation remains authoritative.
- In React, decide deliberately between controlled and uncontrolled inputs and keep submission, pending, success, and error states visible.

## Alternatives & Trade-offs

Native validation is inexpensive and integrates with the browser, but product requirements may need custom error timing or server-error mapping. A validation library can standardise complex forms, but it adds an abstraction that still needs to preserve native semantics and keyboard behaviour.

Controlled React inputs make state and conditional UI explicit. Uncontrolled inputs can be simpler and reduce rerenders, especially for large forms. The right choice depends on how much the UI must react to every change.

## Common Mistakes

- Using placeholder text as the only label.
- Disabling submit controls without explaining what the user must fix.
- Clearing all input after a validation error.
- Treating browser validation as sufficient for server-side data integrity.
- Using a clickable styled element instead of a native submit button.

## Common Interview Questions

### Foundation

- How do you associate a label with an input?
- What is the purpose of `name` on a form control?
- What is the difference between a checkbox and a radio group?

### Intermediate

- How would you display a server-side validation error next to the correct field?
- When would you choose a controlled versus uncontrolled React input?
- How should a form behave while its submission is pending?

### Advanced and Follow-up

- How would you prevent duplicate submissions without making the form unusable with a keyboard?
- How would you handle validation for a form whose fields appear conditionally?

### Code Prediction

Given a form with an input that has no `name`, no associated `label`, and a button outside the form, identify what is missing from submitted data, accessible naming, and native submission behaviour.

## Practical Tasks

- Repair a form with missing labels, incorrect input types, poor error placement, and lost values after submission.
- Design the loading, success, and failure states for a form that calls an ASP.NET Core API.

## Readiness Criteria

You can build a labelled form with appropriate native controls, explain validation boundaries, map API errors to fields, and justify controlled or uncontrolled React state.

## References

- [MDN: Your first form](https://developer.mozilla.org/docs/Learn/Forms/Your_first_form)
- [MDN: Client-side form validation](https://developer.mozilla.org/docs/Learn/Forms/Form_validation)
