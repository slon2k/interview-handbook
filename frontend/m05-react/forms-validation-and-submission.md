# Forms, Validation, and Submission

## Definition

A React form collects user input, displays validation feedback, and submits a deliberate payload to an API. A **controlled** input gets its value from React state and reports every change through an event handler; an **uncontrolled** input keeps its current value in the DOM itself, read only when needed (via a ref or native form submission). Client-side validation gives immediate feedback; server-side validation remains the authoritative check, since a client can always be bypassed.

```tsx
function ControlledInput() {
  const [value, setValue] = useState("");
  return <input value={value} onChange={e => setValue(e.target.value)} />; // React OWNS the value at all times
}
```

## Alternatives & Trade-offs

Controlled inputs let every keystroke trigger application logic immediately — live validation, character counting, conditionally showing other fields — at the cost of a rerender on every keystroke, which can matter for a very large form. Uncontrolled inputs (or a form library built on them, like React Hook Form) avoid that per-keystroke rerender by letting the DOM hold the value until submission, at the cost of needing a ref or the library's own API to read values, rather than having them always available in state.

## How It Works

### Controlled vs. uncontrolled — who owns the current value

```tsx
// Controlled: React state is the source of truth; the input can never show a value React doesn't know about
function Controlled() {
  const [name, setName] = useState("");
  return <input value={name} onChange={e => setName(e.target.value)} />;
}

// Uncontrolled: the DOM owns the value; React reads it only when actually needed
function Uncontrolled() {
  const nameRef = useRef<HTMLInputElement>(null);
  function handleSubmit() { console.log(nameRef.current?.value); }
  return <input ref={nameRef} defaultValue="" />;
}
```

Mixing the two — starting a controlled input's `value` at `undefined` and later giving it a real string — triggers React's "a component is changing an uncontrolled input to be controlled" warning, since the input silently switched modes partway through its lifecycle.

### Keeping a form draft separate from the API payload

```tsx
type OrderFormDraft = { quantity: string; note: string };  // display-friendly: quantity as a STRING while being typed

type CreateOrderRequest = { quantity: number; note: string | null }; // the actual API shape

function toRequest(draft: OrderFormDraft): CreateOrderRequest {
  return { quantity: Number(draft.quantity), note: draft.note.trim() || null };
}
```

A draft's display needs (an empty string while a number field is being typed, before it's a valid number at all) and the API's needs (an actual `number`, `null` instead of an empty string) are often genuinely different shapes — keeping them as two distinct types with an explicit mapping function avoids awkward compromises trying to force one shape to serve both purposes.

### Modeling submission as explicit states

```tsx
type SaveState =
  | { status: "editing" }
  | { status: "submitting" }
  | { status: "validation-error"; fieldErrors: Record<string, string[]> }
  | { status: "conflict"; message: string }
  | { status: "success" };
```

### Mapping server validation errors back to specific fields

```tsx
// ASP.NET Core's ValidationProblemDetails.errors shape: { "Quantity": ["must be greater than 0"] }
function mapValidationErrors(problem: ValidationProblemDetails): Record<string, string[]> {
  return problem.errors ?? {};
}

function FieldError({ fieldErrors, field }: { fieldErrors: Record<string, string[]>; field: string }) {
  const messages = fieldErrors[field];
  return messages ? <span role="alert">{messages.join(", ")}</span> : null;
}
```

`401`/`403` are authentication/authorization problems, not validation failures, and shouldn't be displayed as field errors. A `409` conflict needs its own deliberate recovery path (reload and compare, or an explicit retry) rather than being treated like a retryable field error. See [Module 4: API Contracts](../m04-browser-platform-and-aspnet-core-api-integration/api-dtos-pagination-and-validation-errors.md) and [Auth Status Handling](../m04-browser-platform-and-aspnet-core-api-integration/cookies-storage-auth-and-status-handling.md).

## Application

Use controlled inputs when behavior genuinely depends on every keystroke; use uncontrolled inputs or a form library for large forms where per-keystroke rerenders matter. Keep a form's draft type separate from the API's request type when their shapes genuinely differ, with an explicit mapping function between them. Model submission as explicit states, and map server validation errors to specific fields rather than collapsing them into one generic message.

## Common Mistakes

- Switching an input between controlled and uncontrolled by letting its `value` start as `undefined`, triggering React's mode-switch warning.
- Treating client-side validation as a substitute for server-side validation, when a client can always be bypassed.
- Sending a form's display-formatted draft values directly as the API payload instead of mapping to the actual expected shape.
- Collapsing distinct server failures (`400` validation, `401`/`403` auth, `409` conflict, `500` unexpected) into one generic error message, losing actionable, field-specific feedback.
- Disabling the submit button on failure with no path to retry, or allowing duplicate submissions with no submitting state at all.

## Common Interview Questions

### Basic
- What's the difference between a controlled and an uncontrolled input?
- Why does a form need server-side validation even when it already validates client-side?

### Intermediate
- How would you represent a form's submitting and validation-error states explicitly?
- Why might a form's draft type differ from the API request type it eventually sends?

### Advanced
- How would you map an ASP.NET Core `ValidationProblemDetails` error dictionary onto specific form fields?
- How should `400`, `401`, `403`, `409`, and `500` responses each produce different, deliberate UI behavior in a form?

### Follow-up Questions
- Does disabling the submit button during submission alone prevent duplicate submissions?
- Can a form library like React Hook Form use uncontrolled inputs under the hood while still exposing validation state?

### Code Prediction
```tsx
function Field({ value }: { value?: string }) {
  return <input value={value} onChange={() => {}} />;
}
```
If `value` is `undefined` on the first render and later becomes a real string, what warning does React produce, and why?

## Practical Tasks

- Build a controlled form with local validation and server field-error mapping using a discriminated `SaveState`.
- Compare a controlled and an uncontrolled implementation of the same large form, and justify a choice based on rerender cost.
- Implement distinct recovery behavior for a `409` conflict versus a `400` validation failure in the same form.

## Readiness Criteria

Choose controlled versus uncontrolled inputs deliberately, model submission state explicitly, and map distinct server failure types to distinct, appropriate UI behavior rather than one generic error message.

## References

- [React: Sharing State Between Components](https://react.dev/learn/sharing-state-between-components)
- [React: `useState`](https://react.dev/reference/react/useState)
- [React Hook Form documentation](https://react-hook-form.com/)
