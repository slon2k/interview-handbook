# Designing an Endpoint

## What This Assesses

Given a plain-language requirement, can you design a well-shaped API endpoint — correct HTTP method and status codes, sensible request/response contracts, validation, and error handling — drawing on Module 7 and Module 8 together, without needing to write full implementation code.

## Format and Time Expectations

A short requirement ("design an endpoint for X"), usually followed by "what status codes would this return, and when?" — talking through the contract is usually more important than syntactically perfect code.

## Exercise 1: Cancel an Order

**Problem:** Design an endpoint that lets a customer cancel their own order, if it hasn't shipped yet.

**What a strong answer demonstrates:**
- Method/route: `POST /orders/{id}/cancel` (an action-oriented endpoint, Module 7's REST-principles content, since "cancel" isn't naturally a CRUD verb) rather than `PUT /orders/{id}` with a status field.
- Status codes: `200`/`204` on success, `404` if the order doesn't exist *or* doesn't belong to the caller (Module 12's BOLA-avoidance — deliberately not revealing existence to an unauthorized caller), `409 Conflict` if already shipped (Module 7).
- Idempotency: cancelling an already-cancelled order should succeed harmlessly (idempotent), not error.
- Authorization: verify the order belongs to the calling customer, not just that they're authenticated (Module 12).

**Common mistakes:** Using `DELETE /orders/{id}` (implies actually removing the order, not cancelling it — a semantic mismatch), or returning `500` for the "already shipped" business-rule conflict instead of `409`.

## Exercise 2: List Orders with Filtering and Pagination

**Problem:** Design an endpoint returning a customer's orders, filterable by status and paginated.

**What a strong answer demonstrates:** `GET /orders?status=Shipped&page=2&pageSize=25` with explicit, validated query parameters (Module 7's pagination content) rather than an open-ended query language; a capped maximum `pageSize`; a response shape including pagination metadata (`page`, `pageSize`, `totalCount` or a cursor, depending on offset vs. keyset — Module 7/9's content); returning only the calling customer's own orders by default (again, object-level authorization).

**Common mistakes:** Not capping `pageSize`, letting a client request an enormous page and defeat the point of pagination; forgetting to scope results to the authenticated caller at all, returning *every* customer's orders.

## Exercise 3: Upload a Receipt Image

**Problem:** Design an endpoint accepting an image upload attached to an expense report.

**What a strong answer demonstrates:** `POST /expenses/{id}/receipt` accepting `multipart/form-data`; explicit file-size limits and content-type/magic-byte verification (Module 12's file-upload-security content) rather than trusting the claimed `Content-Type`; a randomly-generated server-side filename for storage (Module 12); a clear response (`201` with a URL to retrieve the uploaded receipt, or similar).

**Common mistakes:** Trusting the client-supplied filename or `Content-Type` header as sufficient validation, or not specifying any size limit at all.

## Readiness Criteria

Choose HTTP methods and status codes precisely for both success and failure/conflict cases, design request/response contracts that are explicit rather than open-ended, and default to considering authorization scope and idempotency for every endpoint design without being prompted.

## References

- [HTTP methods and idempotency (Module 7)](../m07-http-rest-api-design/http-methods-and-idempotency.md)
- [HTTP status codes (Module 7)](../m07-http-rest-api-design/status-codes.md)
- [Pagination, filtering, and sorting (Module 7)](../m07-http-rest-api-design/pagination-filtering-and-sorting.md)
- [OWASP Top 10 and API Security Top 10 (Module 12)](../m12-application-security/owasp-top-10-and-api-security-top-10.md)
- [File-upload security (Module 12)](../m12-application-security/file-upload-security.md)
