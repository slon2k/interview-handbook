# HTTP Status Codes

## Definition

A status code is a three-digit number in an HTTP response indicating the outcome of a request, grouped into five classes: **1xx** informational, **2xx** success, **3xx** redirection, **4xx** client error, **5xx** server error. Choosing the right code lets clients, proxies, and monitoring tools react correctly without parsing the response body.

```
200 OK                    — success, response has a body
201 Created                — success, resource created (usually with a Location header)
204 No Content              — success, no body (e.g., after a DELETE)
400 Bad Request              — client sent invalid data
401 Unauthorized               — missing or invalid authentication
403 Forbidden                    — authenticated, but not allowed
404 Not Found                      — resource doesn't exist
409 Conflict                         — request conflicts with current state
422 Unprocessable Entity               — well-formed but semantically invalid
500 Internal Server Error                — unexpected server failure
503 Service Unavailable                    — server temporarily can't handle the request
```

## Alternatives & Trade-offs

Returning precise, standard status codes lets generic HTTP infrastructure (caches, retry logic, monitoring dashboards, API gateways) behave correctly automatically. Returning `200 OK` for everything and encoding the real outcome only in the response body defeats that — a caching proxy, a client's automatic retry logic, or an uptime monitor all lose the information they need, and every consumer has to parse the body just to know if the call "worked."

## How It Works

### `401` vs. `403` — a common point of confusion

```
401 Unauthorized  — "I don't know who you are" (missing/invalid credentials)
403 Forbidden     — "I know who you are, but you can't do this" (authenticated, not authorized)
```

A request with no auth token to a protected resource should return `401`; a request with a valid token belonging to a user without sufficient permissions should return `403`.

### `409 Conflict` for concurrency and state conflicts

```
PUT /orders/42
{"status": "Shipped"}

409 Conflict
{"error": "Order was already cancelled and cannot be shipped"}
```

`409` signals that the request is valid in isolation but conflicts with the resource's current state — distinct from `400` (malformed/invalid request) or `422` (well-formed but fails business validation).

### `400` vs. `422`

```
400 Bad Request           — malformed JSON, missing required field, wrong data type
422 Unprocessable Entity  — well-formed and complete, but fails a business rule (e.g., end date before start date)
```

Many APIs use only `400` for both cases, which is acceptable, but distinguishing them gives clients more precise information about what kind of validation failed.

### Retry implications by class

```
5xx — often safe to retry (server-side issue, may be transient)
4xx — generally not safe to retry as-is (the request itself is wrong; retrying identically will fail again)
429 Too Many Requests — safe to retry after the delay indicated by a Retry-After header
```

## Application

Choose status codes based on outcome, not convenience: `201` with a `Location` header after creating a resource, `204` after a successful `DELETE` with nothing to return, `404` when a resource genuinely doesn't exist (as opposed to `403` when it exists but access is denied — though some APIs deliberately return `404` instead of `403` to avoid revealing a resource's existence to unauthorized callers), and the correct `4xx`/`5xx` distinction so automated retry logic behaves correctly.

## Common Mistakes

- Returning `200 OK` for every response and encoding real success/failure only in the response body, defeating HTTP-level tooling.
- Confusing `401` and `403` — a very common interview trip-up.
- Returning `500` for validation failures that are actually the client's fault (should be `400`/`422`), which incorrectly signals a server-side bug to monitoring systems.
- Not including a `Retry-After` header with `429`/`503` responses, leaving clients to guess how long to back off.
- Returning `404` for a resource that exists but the caller lacks permission to see, without considering whether that's an intentional security decision (hiding existence) or an accidental misuse of the code.

## Common Interview Questions

### Basic
- What's the difference between `401` and `403`?
- What status code should a successful `DELETE` return?

### Intermediate
- When would you use `409` instead of `400` or `422`?
- Why is it a problem to always return `200 OK` and put the real result in the response body?

### Advanced
- How do retry strategies typically differ between `4xx` and `5xx` responses, and why?
- When, if ever, is it appropriate to return `404` instead of `403` for an authorization failure?

### Follow-up Questions
- What does `429 Too Many Requests` signal, and what header commonly accompanies it?
- Is `422` part of the original HTTP specification, or an extension?

### Code Prediction
A client sends `PUT /orders/42` to cancel an order that has already shipped. The business rule says shipped orders cannot be cancelled. What status code should the response return, and why is it neither `400` nor `500`?

## Practical Tasks

- Given a list of API scenarios (missing auth token, insufficient permissions, resource not found, validation failure, business-rule conflict, unhandled exception), assign the correct status code to each.
- Design the response contract (status code + body shape) for a payment endpoint's failure modes.
- Explain, for a monitoring dashboard, why distinguishing `4xx` from `5xx` rates matters operationally.

## Readiness Criteria

Correctly assign status codes to a range of realistic API outcomes, precisely distinguish the commonly confused pairs (`401`/`403`, `400`/`422`), and explain the operational implications of status-code choice for caching, retries, and monitoring.

## References

### Other

- [MDN: HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
- [RFC 9110: Status Codes](https://www.rfc-editor.org/rfc/rfc9110#name-status-codes)
