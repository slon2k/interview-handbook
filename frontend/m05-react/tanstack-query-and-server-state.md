# TanStack Query and Server State

## Definition

Server state is remote data your application doesn't own outright — it can be stale the moment it's fetched, cached, invalidated, retried, or changed by another user entirely. TanStack Query (and similar libraries) provide a purpose-built lifecycle for exactly this: a **query** fetches and caches data under a stable key; a **mutation** changes remote data and typically invalidates the queries it affects.

```tsx
const { data, isLoading, error } = useQuery({
  queryKey: ["orders", customerId],
  queryFn: () => fetchOrders(customerId),
});
```

## Alternatives & Trade-offs

A hand-written `useEffect` + `fetch` (Module 4's territory) is sufficient for an isolated, short-lived request with no cache reuse and no coordination with other parts of the UI — and adds no extra dependency. TanStack Query becomes worthwhile once several consumers need the same remote data, a mutation needs to update multiple related views consistently, or retries/background-refresh/staleness policy need to be consistent across the app — at the cost of learning its cache/key model and depending on an external library for something a plain `fetch` call could technically do alone.

## How It Works

### The query key must include every input that changes the result

```tsx
// WRONG: keyed only by "orders" — switching customers reuses the WRONG customer's cached data
useQuery({ queryKey: ["orders"], queryFn: () => fetchOrders(customerId) });

// RIGHT: customerId is part of the key, so each customer gets its own independent cache entry
useQuery({ queryKey: ["orders", customerId], queryFn: () => fetchOrders(customerId) });
```

If the key doesn't include `customerId`, navigating from customer 1 to customer 2 can display customer 1's cached orders under the key `"orders"`, since the library has no way to know the request's actual dependency changed — this is precisely analogous to a missing `useEffect` dependency, just at the caching layer instead.

### `isLoading` vs. `isFetching` — initial load versus background refresh

```tsx
const { data, isLoading, isFetching } = useQuery({
  queryKey: ["orders", customerId],
  queryFn: () => fetchOrders(customerId),
});

if (isLoading) return <Spinner />;              // TRUE ONLY on the very first fetch for this key, no cached data at all yet
// isFetching is true during ANY fetch, including a background refresh of already-cached data —
// showing a full-page spinner for isFetching would hide perfectly good existing data unnecessarily
```

### Mutations, and invalidating the queries they affect

```tsx
const queryClient = useQueryClient();

const { mutate: cancelOrder } = useMutation({
  mutationFn: (orderId: string) => cancelOrderRequest(orderId),
  onSuccess: (_, orderId) => {
    queryClient.invalidateQueries({ queryKey: ["orders", customerId] }); // refetch the affected list
  },
});
```

Invalidating a query tells the library "this cached data might now be wrong — refetch it the next time it's needed" — the alternative, manually updating the cached array in place after every mutation, is more precise but far more error-prone to keep consistent across every place that data might be displayed.

### Retrying authorization failures is wrong — they aren't transient

```tsx
useQuery({
  queryKey: ["orders", customerId],
  queryFn: () => fetchOrders(customerId),
  retry: (failureCount, error) => {
    if (error.status === 401 || error.status === 403) return false; // never retry these — they won't resolve themselves
    return failureCount < 3; // do retry genuinely transient failures, a limited number of times
  },
});
```

A default retry policy that blindly retries every failure treats a `401`/`403` (an authorization problem, not a flaky network) the same as a genuinely transient `503` — wasting requests and delaying the point at which the UI actually shows the real, non-retryable problem to the user.

## Application

Reach for a server-state library once more than one part of the UI needs the same remote data, or once mutation-driven invalidation across several views becomes hard to keep consistent by hand. Include every actual input to the request in the query key. Distinguish initial loading from background refresh in the UI. Never retry authorization failures as if they were transient.

## Common Mistakes

- Using an incomplete query key (missing a filter, an ID, or a page number), causing stale or wrong-context data to be served from the cache.
- Showing a full-page loading spinner for every background refetch (`isFetching`) instead of only for the true initial load (`isLoading`), hiding perfectly usable existing data.
- Retrying `401`/`403` responses with the default retry policy, wasting requests on a failure that will never resolve itself.
- Invalidating far more queries than a mutation actually affects, causing unnecessary network traffic.
- Reaching for TanStack Query for a single, isolated request with no reuse or coordination need, adding a dependency for no real benefit.

## Common Interview Questions

### Basic
- What makes server state different from ordinary local UI state?
- What does a query key do, and why does it matter?

### Intermediate
- What's the difference between `isLoading` and `isFetching`, and why does the UI need to distinguish them?
- How does a mutation typically keep related queries up to date after it succeeds?

### Advanced
- Walk through why an incomplete query key can cause the wrong cached data to be displayed after navigating between records.
- When is raw `useEffect`-based fetching (Module 4) genuinely sufficient, and what specific need pushes a team toward a server-state library instead?

### Follow-up Questions
- Should a `401` or `403` response be retried the same way a `503` is?
- Does invalidating a query guarantee the UI shows fresh data immediately, or does it just mark the cache as needing a refetch?

### Code Prediction
```tsx
useQuery({ queryKey: ["orders"], queryFn: () => fetchOrders(customerId) });
```
A user navigates from customer 1's page to customer 2's page. What data might briefly (or persistently) display for customer 2, given this query key, and what's the one-line fix?

## Practical Tasks

- Design query keys and invalidation rules for a paginated, filterable list backed by an ASP.NET Core API.
- Implement a mutation that cancels an order and invalidates exactly the queries it affects, no more and no fewer.
- Configure a retry policy that skips `401`/`403` failures but retries other errors a limited number of times.

## Readiness Criteria

Design correct, complete query keys, distinguish initial loading from background refetching in the UI, and configure mutation invalidation and retry policy deliberately rather than relying on defaults blindly.

## References

- [TanStack Query documentation](https://tanstack.com/query/latest)
- [TanStack Query: Important Defaults](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults)
- [TanStack Query: Query Keys](https://tanstack.com/query/latest/docs/framework/react/guides/query-keys)
