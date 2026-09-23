# State Categories and Application Architecture

## Definition

React applications contain different kinds of state: local UI state, shared client state, server state, and URL state. Architecture becomes clearer when each value has one appropriate owner and one source of truth.

## How It Works

- Local UI state belongs to one component or a small subtree, such as an open menu or draft input.
- Shared client state is used by multiple distant features and is controlled by the client application rather than the server.
- Server state is remote, asynchronous, cacheable, and subject to invalidation or refetching.
- URL state represents navigation, filters, sorting, or pagination that should survive refresh and be shareable.
- Feature-oriented structure groups routes, components, hooks, API calls, and state decisions by user capability instead of technical type alone.

## Application

Before choosing a state library, classify the value. Keep server state in a server-state abstraction when caching and invalidation matter, and do not copy remote data into unrelated global client state without a clear reason.

### State ownership decision process

Ask these questions in order:

1. Is the value authoritative on the server, asynchronous, or cacheable? It is server state.
2. Must the value survive refresh, back/forward navigation, or sharing a link? It is URL state.
3. Is it used by one component or a small subtree? Keep it as local state.
4. Do several distant client-owned features need it? Consider shared client state through composition, context, or a store.

This order prevents a common inversion: selecting a global store first, then forcing every value into it. Form drafts are usually local feature state. A search filter belongs in the URL rather than both a store and query parameters. Remote data should not be copied into a client store merely to make it globally available.

### Feature-oriented organization

Group a feature's route, components, API boundary, hooks, and types together when they change together:

```text
features/products/
  api/products-api.ts
  components/product-list.tsx
  hooks/use-products.ts
  types.ts
  product-routes.tsx
```

This does not prohibit shared components or shared API infrastructure. It makes ownership visible before extracting an abstraction that might be premature.

## Common Mistakes

- Putting every value into a global store.
- Treating server state as ordinary local state and reimplementing caching, retries, and invalidation inconsistently.
- Duplicating URL filters in local state so the two sources drift.
- Storing derived data in multiple places.
- Organizing all files by technical type and making feature behavior hard to discover.
- Putting form drafts in global state when only one feature edits them.
- Mirroring URL state or server data into another state store without choosing one source of truth.

## Common Interview Questions

### Foundation

- What are the main categories of application state?
- What is the difference between client state and server state?

### Intermediate

- Where should a shareable filter live?
- When should state be lifted, placed in context, or kept local?

### Advanced and Follow-up

- Why is server state different enough to justify TanStack Query or an equivalent library?
- How would you split a growing React app into feature-oriented boundaries?

### Code Prediction

Given a search filter stored both in URL parameters and a global store, predict how the values can diverge after browser back navigation and identify the better source of truth.

## Practical Tasks

- Classify the state in a list-and-edit feature as local, shared, server, or URL state.
- Refactor duplicated state so each value has one owner and derived values are calculated from it.
- Defend a state-placement decision for a searchable, paginated list and identify the source of truth for every value.

## Readiness Criteria

You can classify state accurately, choose ownership deliberately, and explain how state categories influence component boundaries and library selection.

## References

- [React: Managing state](https://react.dev/learn/managing-state)
- [React: Sharing state between components](https://react.dev/learn/sharing-state-between-components)
- [TanStack Query documentation](https://tanstack.com/query/latest)
