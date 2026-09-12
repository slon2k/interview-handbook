# Same-Origin Policy, CORS, Caching, and Browser Security

## Definition

Browsers enforce a same-origin policy to restrict how documents from one origin can interact with resources from another. CORS is the mechanism that allows controlled cross-origin requests when both the browser and the server agree. Browser security also includes cookie policies, storage boundaries, and how caching affects behavior.

## How It Works

- A request is same-origin if scheme + host + port match exactly; otherwise it is cross-origin.
- CORS adds response headers such as `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, and `Access-Control-Allow-Headers` to declare allowed cross-origin access.
- Credentials such as cookies or Authorization headers are subject to additional rules and are often blocked unless the server explicitly allows them.
- Browser cache behavior is not the same as server-side caching; the client can cache resources and responses based on headers and policy.
- Security-sensitive mistakes often arise from trusting browser state or assuming the browser will “help” without server cooperation.

## Application

When you debug front-end integration issues, distinguish between browser restrictions, server configuration, and application logic. A failed request can be caused by origin policy, missing headers, cookies, or wrong environment configuration.

## Common Mistakes

- Assuming a server only needs to allow the request from the browser and not the browser's credential rules.
- Forgetting that `fetch` may treat credentials differently depending on `credentials: "include"` or `same-origin`.
- Assuming a cached response means the data is fresh or authorized.
- Treating CORS as a server-side issue only; the browser is the enforcement point.

## Common Interview Questions

### Foundation

- What is the same-origin policy?
- Why does a browser sometimes block a fetch even when the server is reachable?

### Intermediate

- What CORS headers are typically required for a browser request to succeed?
- How do cookies and credentials influence cross-origin requests?

### Advanced and Follow-up

- How would you debug a request that works in Postman but fails in the browser?
- Why can the same backend route behave differently across local development, staging, and production environments?

### Code Prediction

Explain what happens when a frontend running on `http://localhost:3000` calls an API on `https://api.example.com` without matching `Access-Control-Allow-Origin` and credential policy.

## Practical Tasks

- Compare a same-origin request with a cross-origin request and identify the browser behavior in each case.
- Inspect a browser network trace and explain which headers would cause a request to fail or succeed.

## Readiness Criteria

You can explain browser security boundaries, CORS mechanics, and the difference between browser policy enforcement and server-side application logic.

## References

- [MDN: Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [MDN: Fetch API request credentials](https://developer.mozilla.org/en-US/docs/Web/API/Request/credentials)
- [MDN: HTTP caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching)
