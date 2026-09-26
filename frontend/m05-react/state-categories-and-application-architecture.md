# State Categories and Application Architecture

## Definition

React applications hold several genuinely different kinds of state: **local UI state** (owned by one component or small subtree), **shared client state** (used by multiple distant features, owned by the client), **server state** (remote, asynchronous, cacheable, subject to invalidation), and **URL state** (navigation, filters, sorting — should survive refresh and be shareable). Architecture gets clearer once every value has exactly one appropriate owner.

```tsx
const [isMenuOpen, setIsMenuOpen] = useState(false);      // local UI state — belongs to this component alone
const { data: orders } = useQuery({ queryKey: ["orders"] }); // server state — owned by a server-state library
```

## Alternatives & Trade-offs

Defaulting every piece of state to a global store is simple to reach for and never requires deciding where a value "really" belongs, but it makes ownership unclear, triggers unnecessary re-renders across unrelated features, and often duplicates server or URL state that already has a better home. Classifying state by category first — asking what kind of value this actually is before deciding where it lives — costs a moment of deliberate thought per value, but keeps each one owned by exactly the mechanism actually suited to it.

## How It Works

### The state-ownership decision process, applied in order

```
1. Is the value authoritative on the SERVER, asynchronous, or cacheable?          -> server state (TanStack Query)
2. Must it survive refresh, back/forward navigation, or be shareable via a link?   -> URL state (React Router)
3. Is it used by only ONE component or a small subtree?                            -> local state (useState)
4. Do several DISTANT client-owned features genuinely need it?                       -> shared client state (context/store)
```

```tsx
// A search filter answers "yes" to question 2 -> it belongs in the URL, not a global store
const [searchParams, setSearchParams] = useSearchParams();
const filter = searchParams.get("filter") ?? "";
```

Asking these questions *in this order* prevents a common inversion: reaching for a global store first, then trying to force every value (including ones that are really server or URL state) into it, rather than recognizing each value's actual category upfront.

### Why remote data doesn't belong in a plain client store

```tsx
// WRONG: copies server data into a client store, then has to hand-roll caching/invalidation
const useOrdersStore = create<{ orders: Order[] }>(() => ({ orders: [] }));

// RIGHT: server state stays in a server-state library, which already handles caching/invalidation
const { data: orders } = useQuery({ queryKey: ["orders", customerId], queryFn: () => fetchOrders(customerId) });
```

This is exactly the mistake the previous two topics warned about from their own angles — copying remote data into ordinary client state duplicates work a server-state library already does correctly, and creates a second source of truth that can drift from what the server actually has.

### Feature-oriented file organization — grouping by what changes together

```text
features/products/
  api/products-api.ts
  components/product-list.tsx
  hooks/use-products.ts
  types.ts
  product-routes.tsx
```

Grouping a feature's route, components, API calls, hooks, and types together (rather than splitting every project by technical layer — `all-components/`, `all-hooks/`, `all-api-calls/`) makes it obvious which files change together when the feature changes, without prohibiting genuinely shared, cross-feature components or infrastructure from living separately.

## Application

Classify a value's state category *before* deciding where it should live, following the decision process above in order. Keep server state in a server-state library, URL-shareable state in the router, and reserve a global client store for values genuinely needed across several distant, client-owned features — not as a default first choice.

## Common Mistakes

- Putting every piece of state into a global store by default, rather than classifying each value's actual category first.
- Treating server-fetched data as ordinary local or global client state, then hand-rolling caching, retries, and invalidation inconsistently across the app.
- Duplicating a URL-shareable filter into local or global state as well, letting the two sources of truth drift apart after a refresh or back navigation.
- Organizing an entire codebase by technical file type instead of by feature, making it hard to see which files actually change together.

## Common Interview Questions

### Basic
- What are the main categories of state in a React application?
- What's the practical difference between client state and server state?

### Intermediate
- In what order would you evaluate a new piece of state to decide where it belongs?
- Why shouldn't a shareable filter be duplicated in both the URL and a global store?

### Advanced
- Why is server state different enough from client state to justify a dedicated library like TanStack Query, rather than treating it as ordinary state?
- How would you reorganize a codebase split entirely by technical layer into feature-oriented boundaries?

### Follow-up Questions
- Does classifying state by category mean a global client store is never appropriate?
- Can URL state and local component state both legitimately exist for the same feature, for different values?

### Code Prediction
A search filter is stored both as a query parameter and in a global Zustand store, kept "in sync" by an effect. After the user navigates back using the browser's back button, what could happen to these two values, and which one should have been the actual single source of truth?

## Practical Tasks

- Classify each piece of state in a list-and-edit feature as local, shared, server, or URL state, and justify each choice.
- Refactor a value duplicated between local state and a global store into a single source of truth.
- Reorganize a small technical-layer-organized project (all components together, all hooks together) into feature-oriented folders.

## Readiness Criteria

Classify state accurately using the ownership decision process, avoid duplicating server or URL state into a client store, and organize a codebase around features rather than technical layers alone.

## References

- [React: Managing State](https://react.dev/learn/managing-state)
- [React: Sharing State Between Components](https://react.dev/learn/sharing-state-between-components)
- [TanStack Query documentation](https://tanstack.com/query/latest)
