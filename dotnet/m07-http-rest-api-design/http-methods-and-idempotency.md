# HTTP Methods and Idempotency

## Definition

HTTP methods (verbs) express the intent of a request: `GET` retrieves, `POST` creates or triggers a non-idempotent action, `PUT` replaces, `PATCH` partially updates, `DELETE` removes. A method is **idempotent** if making the same request multiple times has the same effect as making it once — important for safe retries over an unreliable network.

```
GET    /orders/42      — retrieve, safe, idempotent
POST   /orders         — create, not idempotent
PUT    /orders/42      — replace entirely, idempotent
PATCH  /orders/42      — partial update, not guaranteed idempotent
DELETE /orders/42      — remove, idempotent
```

## Alternatives & Trade-offs

Using standard HTTP methods correctly lets clients, proxies, caches, and libraries make safe assumptions automatically (e.g., a browser or HTTP client can safely retry a `GET` on a network failure without asking the caller). Ignoring the standard semantics (e.g., using `GET` to trigger a state change) breaks those assumptions silently — caches may prefetch or replay the request, and retry logic built into HTTP clients or proxies may cause unintended duplicate effects.

## How It Works

### Idempotency in practice

```csharp
// PUT is idempotent: calling it once or three times leaves the resource in the same final state
await client.PutAsJsonAsync("https://api.example.com/orders/42", new { status = "Shipped" });
await client.PutAsJsonAsync("https://api.example.com/orders/42", new { status = "Shipped" }); // no additional effect

// POST is not idempotent by default: calling it twice typically creates two resources
await client.PostAsJsonAsync("https://api.example.com/orders", newOrder);
await client.PostAsJsonAsync("https://api.example.com/orders", newOrder); // creates a second order
```

### Why idempotency matters for retries

```csharp
// Safe to retry blindly on a timeout, since PUT is idempotent
try { await client.PutAsJsonAsync(url, payload); }
catch (TaskCanceledException) { await client.PutAsJsonAsync(url, payload); } // repeating has no extra effect

// Retrying a POST blindly on a timeout risks creating a duplicate resource,
// since the first request may have actually succeeded before the timeout was observed
```

### Making `POST` safe to retry: idempotency keys

```
POST /payments HTTP/1.1
Idempotency-Key: 5f3c1e2a-...

{"amount": 100.00}
```

The server stores the result keyed by `Idempotency-Key`; a retried request with the same key returns the original result instead of processing the payment again. This is a common pattern for payment APIs specifically because `POST` isn't idempotent by default but the operation (charging a card) absolutely must not happen twice.

### `PATCH` is not guaranteed idempotent

```
PATCH /counters/1 { "increment": 1 }
```

Applying this twice increments the counter twice — a perfectly valid `PATCH`, but not idempotent, illustrating that `PATCH`'s idempotency depends entirely on what the patch document expresses.

## Application

Choose methods based on the semantics you actually need: `GET` for safe reads, `PUT` when the client sends the full replacement representation, `PATCH` for partial updates, `POST` for creation or actions without a natural idempotent identity, `DELETE` for removal. Use idempotency keys for `POST` operations where duplicate execution would be harmful (payments, order placement).

## Common Mistakes

- Using `GET` to trigger a side effect (e.g., `GET /users/42/delete`), which breaks caching, prefetching, and retry assumptions built into browsers, proxies, and HTTP clients.
- Assuming `PATCH` is always idempotent — it depends entirely on the semantics of the patch payload.
- Blindly retrying a `POST` request after a timeout without an idempotency key, risking duplicate resource creation or duplicate side effects (like a double charge).
- Confusing "idempotent" with "safe" — `DELETE` is idempotent (deleting twice has the same end state as deleting once) but not safe (it changes server state), while `GET` is both idempotent and safe.

## Common Interview Questions

### Basic
- What does it mean for an HTTP method to be idempotent?
- Which standard HTTP methods are idempotent?

### Intermediate
- Why is it risky to blindly retry a failed `POST` request?
- What's the difference between `PUT` and `PATCH`?

### Advanced
- How do idempotency keys make an inherently non-idempotent operation (like a payment) safe to retry?
- Why is `GET` used to trigger a side effect considered a design smell, beyond just "convention"?

### Follow-up Questions
- Is `PATCH` guaranteed to be idempotent by the HTTP specification?
- Are idempotent operations always "safe" (free of side effects)? Give a counterexample.

### Code Prediction
Given a network timeout during a `PUT /orders/42` call where the server actually processed the request successfully before the client's connection dropped, what happens if the client retries the same `PUT` call? What would happen instead if it had been a `POST` without an idempotency key?

## Practical Tasks

- Classify a list of API operations (place order, cancel order, update shipping address, upvote a post, refund a payment) by the correct HTTP method and whether it should be idempotent.
- Design an idempotency-key mechanism for a payment endpoint, including what the server needs to store and for how long.
- Identify and fix an API design that uses `GET` for a state-changing operation.

## Readiness Criteria

Correctly classify HTTP methods by safety and idempotency, explain why idempotency matters for retry logic specifically, and design an idempotency-key mechanism for a non-idempotent operation that must not execute twice.

## References

### Other

- [MDN: HTTP request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)
- [RFC 9110: Method Definitions](https://www.rfc-editor.org/rfc/rfc9110#name-methods)
