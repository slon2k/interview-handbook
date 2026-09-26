# React Router, Routes, and URL State

## Definition

React Router maps browser URLs to React views and provides navigation, route parameters, nested layouts, and query-string state — building directly on the browser history and URL mechanics covered in Module 4. A path parameter (`/orders/:orderId`) identifies a specific resource; a query parameter (`?page=2`) typically represents shareable view state like filters, sorting, or pagination.

```tsx
<Route path="/customers/:customerId" element={<CustomerDetail />} />
// inside CustomerDetail: const { customerId } = useParams();
```

## Alternatives & Trade-offs

Storing filter/sort/pagination state only in local component state is simpler to wire up initially, but that state is lost on refresh, can't be shared via a link, and doesn't participate in browser back/forward navigation — all things users reasonably expect from a URL-driven view. Putting that same state in the URL (query parameters) costs a bit more parsing/defaulting logic at the route boundary, but makes the exact same view bookmarkable, shareable, and correctly restorable, at the cost of needing to keep the URL as the single source of truth rather than accidentally duplicating it in local state too.

## How It Works

### Nested routes — a shared layout, a swappable inner page

```tsx
<Routes>
  <Route path="/customers" element={<CustomersLayout />}>
    <Route index element={<CustomerList />} />
    <Route path=":customerId" element={<CustomerDetail />} />
  </Route>
</Routes>
```

`CustomersLayout` renders shared chrome (a sidebar, a header) once via an `<Outlet />`, while the active child route swaps in and out inside it — navigating between customers never needs to re-mount the shared layout.

### URL-owned view state, parsed and defaulted deliberately at the boundary

```tsx
const [searchParams] = useSearchParams();

const query = searchParams.get("query") ?? "";
const requestedPage = Number(searchParams.get("page") ?? "1");
const page = Number.isInteger(requestedPage) && requestedPage > 0 ? requestedPage : 1; // never trust raw query input
```

A query string is external input a user (or a bookmark, or a shared link) can set to literally anything — `?page=abc` or `?page=-5` are both entirely possible, so the route boundary needs to parse and default every value explicitly, exactly like Module 4's untrusted-input handling for any other external boundary.

### Route changes and stale requests — coordinating with cancellation

```tsx
useEffect(() => {
  const controller = new AbortController();
  loadCustomer(customerId, controller.signal).then(setCustomer);
  return () => controller.abort(); // if customerId changes again before this resolves, the OLD request is cancelled
}, [customerId]);
```

When a route parameter changes (navigating from customer 1 to customer 2), any in-flight request for the *previous* parameter is now stale — without cancellation, a slow response for customer 1 could arrive after customer 2's data has already loaded and overwrite it. See [Module 4: Fetch and Cancellation](../m04-browser-platform-and-aspnet-core-api-integration/fetch-requests-cancellation-and-stale-responses.md).

### Router-aware navigation, not plain anchors

```tsx
import { Link, useNavigate } from "react-router-dom";

<Link to={`/customers/${customerId}`}>View customer</Link> // client-side navigation, no full page reload

function handleSave() {
  navigate(`/customers/${customerId}`); // programmatic navigation after an action completes
}
```

A plain `<a href="...">` would trigger a full page reload, discarding all client-side state and defeating the point of a single-page application — router-provided navigation components keep transitions client-side.

## Application

Use path parameters for resource identity and query parameters for shareable, bookmarkable view state (filters, sort, pagination). Parse and default every query value explicitly at the route boundary rather than trusting it directly. Coordinate route changes with request cancellation so a stale response for a previous route can never overwrite the current one. Use router-provided navigation, not plain anchors, for in-app links.

## Common Mistakes

- Storing shareable filter/sort/pagination state only in local component state, losing it on refresh and breaking back/forward navigation.
- Reading a query parameter directly without parsing or defaulting it, trusting a `?page=` value that could be malformed or missing.
- Using plain `<a href>` anchors for in-app navigation instead of router-aware links, triggering unwanted full-page reloads.
- Letting a stale request for a previous route parameter overwrite the current view's data because no cancellation was wired up.
- Copying query parameters into local state as a second source of truth, letting the two drift apart after a refresh or back navigation.

## Common Interview Questions

### Basic
- When should a value be a path parameter versus a query parameter?
- What does a nested route layout provide that a flat route list doesn't?

### Intermediate
- Why must query parameters be parsed and defaulted rather than trusted directly?
- Why should a route change coordinate with request cancellation?

### Advanced
- How would you preserve a search/filter state across a refresh and browser back navigation without duplicating it in local state?
- Where should route-level authorization and not-found handling live in a nested route structure?

### Follow-up Questions
- Does using `<Link>` instead of `<a>` change what actually appears in the browser's address bar?
- Should a new search query normally reset the page parameter back to 1?

### Code Prediction
Given the URL `/customers/42?tab=orders&page=2`, which values represent resource identity and which represent view state? Which of these should survive a page refresh, and does a plain URL alone guarantee that?

## Practical Tasks

- Design nested routes for a paginated, filterable ASP.NET Core-backed resource with a shared layout.
- Add route-aware request cancellation so navigating away from a record before its data loads can't overwrite the newly selected record.
- Parse malformed or missing query parameters to safe defaults, and verify the resulting view survives a refresh and back navigation correctly.

## Readiness Criteria

Explain route identity, nested layouts, and URL-owned state precisely, parse query input defensively, and coordinate route changes with request cancellation to avoid stale-data bugs.

## References

- [React Router: Main Concepts](https://reactrouter.com/home)
- [React Router: Nested Routes](https://reactrouter.com/start/framework/routing)
- [MDN: URLSearchParams](https://developer.mozilla.org/docs/Web/API/URLSearchParams)
