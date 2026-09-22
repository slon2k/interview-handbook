# Module 4 - Browser Platform and ASP.NET Core API Integration

**Status:** In progress  
**Priority:** High  
**Prerequisites:** [Module 2 - JavaScript Language and Runtime](../m02-javascript-language-and-runtime/README.md) and [Module 3 - TypeScript](../m03-typescript/README.md)

## Scope

This module covers the browser as a runtime boundary: how events flow through the DOM, how the page is painted and debugged, how `fetch` moves data between client and server, and how the browser enforces origin, credential, and storage rules. It also connects those browser realities to practical ASP.NET Core API contracts: DTOs, pagination metadata, validation errors, and authorization responses.

The focus is on the frontend engineer's mental model of the browser and the API boundary. This is where JavaScript, TypeScript, and backend contracts meet in real applications.

## Why This Matters in Interviews

Interviewers commonly ask candidates to explain why a request fails, why a handler runs unexpectedly, why a stale result appears, or why a browser blocks a request before it reaches the server. Strong candidates can reason from browser behavior, HTTP basics, and API boundary design without mistaking client-side rules for server-side rules.

## Learning Outcomes

By the end of this module, you should be able to:

- Explain event bubbling, capturing, delegation, and how DOM handlers interact with form and click events.
- Diagnose common browser rendering issues and explain the relationship between layout, paint, and developer tools.
- Use `fetch`, `AbortController`, request cancellation, and error handling without creating stale or duplicate work.
- Send multipart form data safely and choose an appropriate browser mechanism for realtime updates.
- Explain same-origin, CORS, cookies, local storage, and credential behavior at the browser boundary.
- Model API responses, pagination metadata, validation errors, and authorization failures in a client-friendly way.
- Explain how client-side routing and URL state differ from server-rendered navigation.
- Choose between an API wrapper, a generated client, and a handwritten boundary based on project needs.
- Recognize when frontend code is failing because of browser rules versus backend behavior versus application logic.

## Topics

### 1. Browser Behavior and Rendering

- [DOM events, bubbling, capturing, and delegation](dom-events-bubbling-and-delegation.md)
- [Browser rendering, layout, repaint, and developer tools](browser-rendering-layout-repaint-and-dev-tools.md)

### 2. Network Requests and Browser Boundaries

- [Fetch, cancellation, error handling, and stale responses](fetch-requests-cancellation-and-stale-responses.md)
- [FormData, file uploads, and multipart requests](formdata-file-uploads-and-multipart-requests.md)
- [Same-origin policy, CORS, caching, and browser security](same-origin-policy-cors-caches-and-browser-security.md)
- [Cookies, storage, auth flows, and `401`/`403` handling](cookies-storage-auth-and-status-handling.md)
- [WebSockets, Server-Sent Events, and realtime updates](websockets-server-sent-events-and-realtime-updates.md)

### 3. Routing, State, and API Contracts

- [Client-side routing, URLs, and history state](client-side-routing-urls-and-history-state.md)
- [API DTOs, pagination, and validation errors](api-dtos-pagination-and-validation-errors.md)

### 4. Integration and Tooling

- [OpenAPI/Swagger clients vs. handwritten clients](openapi-swagger-clients-vs-handwritten-clients.md)
- [Environment variables, API proxying, and local development configuration](environment-variables-proxying-and-local-dev.md)

## Scope Boundaries

- JavaScript values, closures, async execution, and event-loop behavior belong in [Module 2](../m02-javascript-language-and-runtime/README.md).
- TypeScript types, DTO modeling, and runtime validation belong in [Module 3](../m03-typescript/README.md); this module applies validated data to HTTP status handling, browser request behavior, and API contracts.
- React rendering, hooks, performance tuning, and component state belong in [Module 5](../m05-react/README.md); this module explains the browser behaviors those components use.
- HTTP semantics, protocol design, and server-side API behavior remain in [Module 7 - HTTP REST API Design](../../dotnet/m07-http-rest-api-design/README.md).
- Security theory, threat modeling, and backend authorization architecture belong in [Module 12 - Application Security](../../dotnet/m12-application-security/README.md).
- Browser performance and observability concerns sit in [Module 13 - Performance Diagnostics and Observability](../../dotnet/m13-performance-diagnostics-observability/README.md).

## Suggested Learning Sequence

1. Learn the browser event model and the rendering pipeline before debugging UI behavior.
2. Understand how `fetch`, cancellation, and network errors work before writing API wrappers.
3. Study origin, credentials, cookies, and client-side storage to explain browser security decisions.
4. Model client-side routing and URL state so the app remains predictable and shareable.
5. Send form data and files deliberately, then choose an HTTP, WebSocket, or SSE contract for a feature's update direction.
6. Map API contracts and validation failures into known frontend states.
7. Finish with local development setup, generated clients, and environment-driven integration choices.

## Practical Deliverables

- Explain why a click handler fires for a nested element and rewrite the handler using event delegation.
- Build a `fetch` wrapper that handles cancellation, revalidation, and stale request races.
- Diagnose a CORS or cookie problem and explain whether the failure is on the browser or the server.
- Model a paginated ASP.NET Core response and display field-level validation errors without silently dropping information.
- Submit a file with `FormData`, show upload progress limitations, and handle a structured validation error.
- Choose polling, Server-Sent Events, or WebSockets for a realtime requirement and justify the trade-off.
- Decide whether a project should use a generated OpenAPI client or a handwritten API boundary and justify the choice.

## Interview Coverage

Each topic includes foundation, intermediate, and advanced questions plus a code-prediction or debugging prompt. Practice explaining what happens at runtime, which layer causes the bug, and how to make the system more robust without guessing.

## References

- [MDN: EventTarget.addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [MDN: Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)
- [MDN: History API](https://developer.mozilla.org/en-US/docs/Web/API/History)
- [ASP.NET Core: Handle errors in ASP.NET Core web APIs](https://learn.microsoft.com/aspnet/core/web-api/handle-errors)
- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
