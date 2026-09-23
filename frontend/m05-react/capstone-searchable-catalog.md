# Guided Capstone: Searchable Catalog

## Goal

Build a React and TypeScript catalog backed by an ASP.NET Core API. The feature must support accessible search and filters, URL-owned pagination, cancellable requests, a detail route, and an edit form that handles server validation and conflict responses.

This is a guided implementation exercise, not a reference solution. Use the linked modules to make each decision and explain it as you would in an interview.

## Requirements

- Render a semantic search form with a labelled input, submit button, and clear action.
- Keep `query`, `category`, and `page` in the URL. Invalid or absent values must resolve to safe defaults.
- Request a typed paginated catalog endpoint when the URL state changes.
- Cancel outdated requests and never let an old result replace a newer query.
- Render distinct initial-loading, background-refresh, success, empty, retryable-error, unauthorized, and forbidden states.
- Navigate to `/products/:productId` for a detail view.
- Submit an edit form that preserves user input and maps `ValidationProblemDetails.errors` to fields.
- Give `401`, `403`, `409`, and unexpected server failures distinct recovery behavior.

## Suggested Contract

Use a contract equivalent to the following. Treat received JSON as untrusted until the Module 3 API boundary validates it.

```typescript
type ProductDto = {
  id: string;
  name: string;
  category: string;
  price: number;
};

type PagedResult<T> = {
  items: T[];
  page: number;
  pageSize: number;
  totalCount: number;
};

type ProductQuery = {
  query: string;
  category: string | null;
  page: number;
};
```

## Checkpoint 1: Accessible Search and Layout

Create a semantic search form and product list structure. Use a real `<label>`, `<input>`, `<button>`, list or table semantics appropriate to the data, and visible focus styles. Do not replace controls with clickable generic elements.

Review prompts:

- Which output semantics come from [Module 1](../m01-web-platform-foundations/semantic-html-and-document-structure.md)?
- Which component props are required, optional, or mutually exclusive?

## Checkpoint 2: Typed URL State

Parse the route query string into `ProductQuery`. On a new search, reset `page` to `1`; on a filter change, preserve only values that remain valid. Derive the view from URL state rather than copying filters into a global store.

Review prompts:

- Which values are resource identity and which are shareable view state?
- How does malformed `page=zero` become a safe page value?

References: [React Router and URL state](react-router-routes-and-url-state.md) and [Module 4 URL mechanics](../m04-browser-platform-and-aspnet-core-api-integration/client-side-routing-urls-and-history-state.md).

## Checkpoint 3: API Boundary and Cancellable List Fetch

Create a typed API function that receives `ProductQuery` and an `AbortSignal`. Validate response data at the Module 3 boundary, then start a request whenever URL query inputs change. Abort the old request in effect cleanup and treat `AbortError` as cancellation, not an error state.

Review prompts:

- What prevents a request for `query=book` from overwriting the later `query=books` result?
- Why must `response.ok` be checked even when `fetch` resolves?

References: [typed async UI states](typed-data-fetching-and-async-ui-states.md), [Module 3 runtime validation](../m03-typescript/api-dtos-and-runtime-validation.md), and [Module 4 fetch behavior](../m04-browser-platform-and-aspnet-core-api-integration/fetch-requests-cancellation-and-stale-responses.md).

## Checkpoint 4: Honest UI States

Use a discriminated union to represent initial loading, success, empty success, refresh, error, unauthorized, and forbidden states. Retain prior results during refresh only if the product experience remains clear about the update. Add a retry action that repeats only the intended request.

Acceptance criteria:

- An empty successful response is not presented as an error.
- A cancelled request does not show an error message.
- A `401` activates the sign-in flow; a `403` shows a permission outcome.

## Checkpoint 5: Detail Route and Edit Form

Add a product detail route and cancel its previous request when `productId` changes. Build an edit form with a typed draft distinct from the submit DTO. Preserve the draft on `400` validation errors, render every field message from `ValidationProblemDetails.errors`, and choose a recovery path for a `409` conflict.

Review prompts:

- Why is a server validation error different from `401`, `403`, `409`, and `500`?
- Which state belongs to the form, route, server cache, or URL?

References: [forms, validation, and submission](forms-validation-and-submission.md) and [Module 4 API contracts](../m04-browser-platform-and-aspnet-core-api-integration/api-dtos-pagination-and-validation-errors.md).

## Checkpoint 6: Architecture Review

Organize the feature by ownership, for example `features/products/api`, `components`, `hooks`, and `types`. Extract a custom hook only when it gives a clear typed API. Use a reducer for related transitions and context only for genuinely shared feature concerns.

Optional extension: replace the effect-based list and detail fetches with TanStack Query. Define complete query keys for `ProductQuery`, preserve the API validation boundary, and describe which cache and invalidation responsibilities the library now owns.

## Final Review

You are ready when you can explain:

- how semantic HTML, typed props, URL state, API DTOs, cancellation, and async states combine in one feature;
- why every value has one source of truth;
- how the feature responds differently to empty, validation, authorization, conflict, network, and server failures;
- when TanStack Query would improve this design and when raw fetching remains adequate.

## References

- [Module 5: State categories and application architecture](state-categories-and-application-architecture.md)
- [Module 5: TanStack Query and server state](tanstack-query-and-server-state.md)
- [Module 4: API DTOs, pagination, and validation errors](../m04-browser-platform-and-aspnet-core-api-integration/api-dtos-pagination-and-validation-errors.md)
