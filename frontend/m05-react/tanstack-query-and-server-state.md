# TanStack Query and Server State

## Definition

Server state is remote data that can be loading, stale, cached, invalidated, retried, or changed by another user or process. TanStack Query and similar libraries provide primitives for managing that lifecycle separately from local component state.

## How It Works

- A query is identified by a stable key and retrieves data through a query function.
- The library can cache results, track freshness, deduplicate requests, retry failures, and expose loading or error state.
- Mutations change remote data and commonly invalidate or update affected queries.
- Query keys must include the inputs that change the result, such as a route ID or filter.
- The library does not replace API validation, authorization handling, or thoughtful UI state design.

## Application

Use a server-state library when multiple features need consistent caching, refetching, invalidation, retries, or background refresh. Keep query keys and mutation invalidation rules close to the feature that owns the data contract.

### When raw fetching is enough

An effect-based request can be appropriate for an isolated, short-lived request with no cache reuse, mutation coordination, or background refresh. A server-state library becomes worthwhile when several consumers need the same remote data, a mutation must update related views, retries and freshness need a consistent policy, or the feature needs cached data while refetching.

Do not choose TanStack Query merely to avoid understanding `fetch`, cancellation, or API contracts. It centralizes cache and request lifecycle concerns; it does not validate untrusted JSON, decide how `401` or `403` should behave, or replace local UI state.

## Common Mistakes

- Using an incomplete query key and returning data for the wrong route or filter.
- Treating cached data as permanently fresh.
- Retrying authorization failures as if they were temporary network errors.
- Invalidating every query after every mutation and creating unnecessary traffic.
- Assuming a server-state library removes the need for runtime response validation.
- Using TanStack Query for an isolated request while duplicating its state in component and global stores.

## Common Interview Questions

### Foundation

- What makes server state different from local UI state?
- What problem does a query cache solve?

### Intermediate

- What belongs in a query key?
- How should a mutation update or invalidate related data?

### Advanced and Follow-up

- When is raw `useEffect` fetching sufficient, and when does it become limiting?
- How would you handle stale cached data while showing a responsive UI?

### Code Prediction

Given a query keyed only by `"orders"` while the request also depends on `customerId`, predict what can happen when the user navigates between customers.

## Practical Tasks

- Design query keys and invalidation rules for a paginated ASP.NET Core resource.
- Compare a custom effect-based fetch hook with TanStack Query and identify which lifecycle responsibilities the library removes.
- Decide whether an isolated settings fetch and a shared paginated resource need a server-state library, and justify the difference.

## Readiness Criteria

You can explain server state, cache freshness, stable query keys, mutation invalidation, retries, and the limits of a server-state library.

## References

- [TanStack Query documentation](https://tanstack.com/query/latest)
- [TanStack Query: Important defaults](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults)
- [TanStack Query: Query keys](https://tanstack.com/query/latest/docs/framework/react/guides/query-keys)
