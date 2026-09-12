# Fetch, Cancellation, Error Handling, and Stale Responses

## Definition

`fetch` is the browser API for sending HTTP requests from JavaScript. It supports request/response handling, cancellation, and structured error handling, but it does not automatically make a request “safe” or “finished” in the way a UI state model might expect.

## How It Works

- `fetch(url)` returns a promise that resolves to a `Response` object when the HTTP transport succeeds.
- HTTP status codes such as `404` and `500` are not rejected by `fetch` by default; they are treated as a successful response object unless the network fails.
- `AbortController` allows a request to be cancelled when a component is unmounted or a newer request supersedes it.
- A stale-response problem occurs when an older request resolves after a newer one and mutates UI state incorrectly.
- A robust client checks `response.ok`, handles network errors, and tracks request identity for cancellation or stale-state prevention.

## Application

Use fetch wrappers that centralize status checking, request cancellation, and error mapping at the API boundary. This makes data loading more predictable and reduces subtle race conditions in the UI.

## Common Mistakes

- Treating every `fetch` promise as a success and forgetting to check `response.ok`.
- Not cancelling requests during component unmount or rapid re-fetching.
- Updating state from an old request after a newer one has already succeeded.
- Ignoring `AbortError` and allowing the UI to show a misleading failure message.

## Common Interview Questions

### Foundation

- Why does `fetch` not reject on `404` or `500` by default?
- What is `AbortController` used for?

### Intermediate

- How would you prevent a stale request from overwriting newer data?
- What is the difference between a network error and an HTTP error response?

### Advanced and Follow-up

- How would you structure a `useEffect` fetch so that a cancelled request does not update stale state?
- What is the downside of silently retrying network requests without considering user intent or rate limits?

### Code Prediction

Given two fetches for the same list, where the second starts later, explain which result should win and how you would guard against a stale response.

## Practical Tasks

- Build a fetch wrapper that throws on non-OK responses and handles `AbortError` as a cancellation, not a failure.
- Add request identity to a list page so early results cannot overwrite newer data.

## Readiness Criteria

You can explain `fetch` semantics, HTTP status handling, cancellation, and stale-response patterns well enough to diagnose real UI bugs and design safer client-side data loading.

## References

- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [MDN: AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)
- [MDN: `Response.ok`](https://developer.mozilla.org/en-US/docs/Web/API/Response/ok)
