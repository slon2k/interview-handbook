# Mock Service Worker and API Mocking

## Definition

Mock Service Worker (MSW) intercepts requests at the network boundary and returns controlled responses. Components and API clients continue to use `fetch` or their normal request library, so tests exercise request construction, response parsing, loading states, and error handling without a live backend.

## Alternatives & Trade-offs

Mocking a client method is simpler for a focused unit test, but it can hide an incorrect URL, verb, request body, or response-handling path. MSW gives more realistic integration confidence at the cost of maintaining explicit handlers. Use a client-method mock only when the client itself is outside the behavior under test.

## How It Works

Define reusable default handlers and override a handler for a test scenario:

```ts
server.use(
  http.get("/api/products", () =>
    HttpResponse.json({ items: [], totalCount: 0 }),
  ),
);
```

Handlers should represent the real contract, including successful data, empty results, `ProblemDetails`, validation errors, unauthorized responses, and delays used intentionally to expose pending UI. Reset overrides after each test so one scenario cannot affect another.

## Application

Use MSW for rendered pages and component integrations that make requests. Share contract-shaped handlers between local development and tests only when doing so remains clear; tests still need scenario-specific overrides. Keep backend contract tests on the backend rather than assuming a frontend mock proves server behavior.

## Common Mistakes

- Mocking `fetch` globally and asserting call order instead of testing the rendered outcome.
- Returning response shapes the production API never emits.
- Allowing handlers to leak between tests.
- Testing authorization only as a status code instead of testing the user-visible sign-in or access-denied behavior.

## Common Interview Questions

### Foundation

- What does request-level mocking provide that a mocked function does not?
- Why should an MSW handler resemble the real API contract?

### Intermediate

- How would you test a `ProblemDetails` validation response?
- Why must handlers be reset after each test?

### Advanced and Follow-up

- What confidence does MSW provide, and what still requires backend or E2E testing?

### Code Prediction

If a component changes its request URL but a client-method mock still resolves successfully, why might the test remain green while an MSW test fails?

## Practical Tasks

- Create MSW handlers for success, empty, validation-error, and server-error list responses.
- Test that a page renders a structured validation error without exposing transport details.

## Readiness Criteria

You can choose request-level mocking for API-connected UI, create isolated contract-shaped scenarios, and explain its confidence boundary.

## References

- [MSW documentation](https://mswjs.io/docs/)
- [MSW request handlers](https://mswjs.io/docs/basics/response-resolver/)
