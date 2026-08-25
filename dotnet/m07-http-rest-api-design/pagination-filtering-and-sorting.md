# Pagination, Filtering, and Sorting

## Definition

Pagination limits how many results an endpoint returns per call, splitting a large result set into pages. Filtering lets a client narrow results by criteria; sorting lets a client control result order. All three are contract-design concerns: how a client expresses "which page, filtered how, sorted how" through query parameters or a request body.

```
GET /orders?status=Shipped&sort=-createdAt&page=2&pageSize=25
```

## Alternatives & Trade-offs

**Offset-based pagination** (`page`/`pageSize` or `offset`/`limit`) is simple to implement and lets clients jump to an arbitrary page, but its performance degrades on large offsets (the database still has to scan and discard all skipped rows) and it's vulnerable to items shifting between pages if the underlying data changes between requests. **Keyset (cursor-based) pagination** (`?after=<lastId>`) scales better for large datasets and is stable under concurrent inserts/deletes, at the cost of not supporting "jump to page 50" — only "next" and "previous" relative to a cursor.

## How It Works

### Offset-based pagination

```
GET /orders?page=2&pageSize=25

{
  "items": [...],
  "page": 2,
  "pageSize": 25,
  "totalCount": 143
}
```

Simple to implement (`Skip((page-1)*pageSize).Take(pageSize)` in LINQ terms) but `totalCount` requires a separate count query, and pages can "shift" if a row is inserted or deleted between requests — a user might see the same item twice, or miss one entirely, across two page requests.

### Keyset (cursor) pagination

```
GET /orders?after=142&pageSize=25

{
  "items": [...],
  "nextCursor": "167"
}
```

Instead of an offset, the client passes the last-seen item's identifier (or a sort key); the server queries `WHERE id > @after ORDER BY id LIMIT @pageSize`. This avoids the "scan and discard" cost of large offsets and stays stable even if rows are inserted/deleted, at the cost of not being able to jump directly to an arbitrary page number.

### Filtering — keep the contract explicit

```
GET /orders?status=Shipped&customerId=7&createdAfter=2026-01-01
```

Each filterable field should be an explicit, documented query parameter with a clear type — avoid a single free-text `?query=` parameter unless the endpoint genuinely implements a search feature, since explicit parameters are far easier to validate, document (see `openapi-and-api-contracts.md`), and reason about than an implicit query language.

### Sorting — support explicit direction

```
GET /orders?sort=-createdAt        — descending by createdAt (leading '-' convention)
GET /orders?sort=customerName,-total — sort by customerName ascending, then total descending
```

## Application

Use offset-based pagination for smaller, less frequently-changing datasets or UIs that genuinely need page-number navigation. Use keyset pagination for large, high-churn datasets (activity feeds, logs, high-volume tables) where offset performance and consistency under concurrent writes matter. Always make filtering and sorting parameters explicit and documented rather than an ad hoc query string.

## Common Mistakes

- Using offset-based pagination on a very large table without realizing `Skip(100000)` still has to scan and discard 100,000 rows in most database engines.
- Not accounting for data shifting between offset-paginated requests, causing duplicate or skipped items in a client that pages through results sequentially.
- Building an unbounded, ad hoc filtering system that accepts arbitrary field/operator combinations, which is hard to validate, document, or optimize with indexes.
- Returning an unbounded result set with no pagination at all "because the list is usually small" — a common source of production incidents once the assumption stops holding.
- Forgetting to cap `pageSize` server-side, letting a client request an enormous page and defeat the purpose of pagination entirely.

## Common Interview Questions

### Basic
- What's the difference between offset-based and keyset (cursor-based) pagination?
- Why should an API always cap the maximum page size?

### Intermediate
- Why does offset-based pagination perform poorly on large datasets?
- What consistency problem can occur with offset pagination if rows are inserted between page requests?

### Advanced
- How would you design a keyset pagination scheme for a multi-column sort (e.g., sort by `createdAt`, then `id` as a tiebreaker)?
- How would you support both stable pagination and "total count" without paying for an expensive count query on every request?

### Follow-up Questions
- Can keyset pagination support jumping directly to an arbitrary page number?
- Should filtering and sorting parameters be validated against an allow-list, or accepted freely?

### Code Prediction
A client pages through `GET /orders?page=1` then `page=2` using offset pagination, while another process deletes an order from page 1 between the two requests. What's the observable effect on the client's view of `page=2`'s contents, and how does keyset pagination avoid it?

## Practical Tasks

- Implement both offset-based and keyset-based pagination for the same endpoint and compare their behavior under concurrent inserts.
- Design an explicit, validated filtering and sorting contract for an orders endpoint, including an allow-list of sortable fields.
- Identify the point at which offset-pagination performance would become a real problem for a given table size, and propose a migration to keyset pagination.

## Readiness Criteria

Explain the trade-offs between offset and keyset pagination precisely, design explicit and validated filtering/sorting contracts, and recognize when an unpaginated or offset-paginated endpoint will become a real production problem.

## References

### Other

- [Cursor-based pagination (general pattern reference)](https://relay.dev/graphql/connections.htm)
