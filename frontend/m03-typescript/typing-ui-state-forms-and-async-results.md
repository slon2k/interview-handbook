# Typing UI State, Forms, Callbacks, and Async Results

## Definition

UI types should represent the states and events a feature can *actually* have — empty or populated data, editing or submitting forms, pending/successful/empty/failed asynchronous work — using the union and discriminant modeling this module already established. A well-designed model makes an invalid combination of states genuinely impossible to construct, rather than merely unlikely.

```typescript
type AsyncState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: string };
```

## Alternatives & Trade-offs

Modeling async state as `{ data?: T; loading: boolean; error?: string }` is quick to write and instantly familiar, but permits combinations no real request can ever produce — `loading: true` with a populated `error` at the same time — forcing every consumer to defensively guard against states that should be structurally impossible. A generic discriminated union like `AsyncState<T>` costs a bit more type definition upfront, but the compiler itself then guarantees every consumer only reads `data` where it's actually available, eliminating that whole category of defensive guard entirely.

## How It Works

### A generic result type preserves the data type through every branch

```typescript
type AsyncState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "empty" }
  | { status: "error"; error: string };

function render(state: AsyncState<string[]>): string {
  switch (state.status) {
    case "idle": return "";
    case "loading": return "Loading...";
    case "empty": return "No results";
    case "error": return state.error;
    case "success": return state.data.join(", "); // `data` is string[] HERE — preserved from the generic parameter
  }
}
```

`AsyncState<T>` is reused for any data shape the application fetches — `AsyncState<User[]>`, `AsyncState<Order>` — while each specific instantiation keeps its own `data` field correctly, precisely typed for that particular use.

### Typing callbacks explicitly, instead of `Function` or implicit `any`

```typescript
type OnSave = (draft: OrderDraft) => Promise<void>; // explicit: what it receives, what it returns

function OrderForm({ onSave }: { onSave: OnSave }) { /* ... */ }
```

```typescript
// WRONG: Function accepts absolutely any callable with any signature — no actual safety at all
function OrderForm({ onSave }: { onSave: Function }) { }
```

`Function` as a type is barely more useful than `any` for a callback — it tells the compiler nothing about what arguments the callback expects or what it returns, so a caller could pass a completely incompatible function and nothing would catch it.

### Separating a form's draft model from the eventual API payload

```typescript
type OrderDraft = { quantity: string; note: string };   // display-friendly: quantity is a STRING while being typed
type CreateOrderRequest = { quantity: number; note: string | null }; // the actual API shape

function toRequest(draft: OrderDraft): CreateOrderRequest {
  return { quantity: Number(draft.quantity), note: draft.note.trim() || null };
}
```

A form's draft often needs a looser, display-oriented shape (a number field held as a string while being typed, before it's necessarily valid at all) than the API ultimately expects — keeping them as two distinct types with an explicit mapping function avoids forcing one shape to awkwardly serve both purposes.

### A submission model with explicit, mutually exclusive states

```typescript
type SaveState =
  | { status: "editing" }
  | { status: "submitting" }
  | { status: "validation-error"; fieldErrors: Record<string, string[]> }
  | { status: "success" };
```

Each state's specific fields (`fieldErrors` only in `validation-error`) are available exactly where the discriminant guarantees they exist — the same discriminated-union discipline from earlier in this module, now applied specifically to a form's lifecycle.

## Application

Type the state transitions of a form or data view before implementing any handlers — doing so surfaces missing transitions and prevents success-only assumptions early. Keep this framework-independent type model here; [Module 4's fetch lesson](../m04-browser-platform-and-aspnet-core-api-integration/fetch-requests-cancellation-and-stale-responses.md) covers request cancellation and stale-response handling, and [Module 5's async UI lesson](../m05-react/typed-data-fetching-and-async-ui-states.md) applies these exact states during React rendering.

## Common Mistakes

- Combining independent booleans (`loading`, `error`) that together permit impossible states, such as loading being true alongside a populated error.
- Reusing a server DTO directly as a form's draft model, forcing awkward compromises where the two shapes' actual needs genuinely differ.
- Typing a callback prop as `Function` or leaving it implicitly `any`, losing all safety on what it accepts and returns.
- Making every field of a UI state model optional to avoid explicitly modeling a genuine empty state.

## Common Interview Questions

### Basic
- Why are discriminated unions useful for modeling UI state specifically?
- How should a form's draft model differ from an API DTO?

### Intermediate
- How would you type loading, success, empty, and error states for a list component?
- What should a callback prop's type actually specify, beyond just "it's a function"?

### Advanced
- How would you prevent a submit handler from ever receiving a draft that isn't yet valid for the API payload it maps to?
- Walk through why a generic `AsyncState<T>` preserves precise typing across every branch, for any data shape it's instantiated with.

### Follow-up Questions
- Does a discriminated union for async state need a separate `empty` case, or can that be folded into `success`?
- Should a form's draft type and its API request type ever be the exact same type?

### Code Prediction
```typescript
type AsyncState<T> = { status: "loading" } | { status: "success"; data: T } | { status: "error"; error: string };
function show(state: AsyncState<number>) {
  if (state.status === "success") {
    return state.data;
  }
}
```
Inside the `if` branch, what is the type of `state.data`? What would happen if this same access were attempted outside the `if` block?

## Practical Tasks

- Replace a collection of independent UI booleans with a single discriminated `AsyncState<T>` union.
- Model a form's draft, validation-error, submitting, and success states without any non-null assertions.
- Design and implement a mapping function converting a form draft into its corresponding API request type.

## Readiness Criteria

Model UI state transitions honestly using discriminated unions, carry precise data types through async branches and callbacks, and keep form drafts distinct from API payloads where their actual shapes genuinely differ.

## References

- [TypeScript Handbook: Discriminated Unions](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions)
- [TypeScript Handbook: Function Type Expressions](https://www.typescriptlang.org/docs/handbook/2/functions.html#function-type-expressions)
