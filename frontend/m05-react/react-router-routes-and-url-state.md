# React Router, Routes, and URL State

## Definition

React Router maps browser URLs to React views and provides navigation, route parameters, nested layouts, and query-string state. It builds on the browser history and URL concepts covered in Module 4.

## How It Works

- A route configuration maps a path pattern to an element or route component.
- Nested routes let a parent layout render shared UI while a child route renders the active page.
- Route parameters identify a resource such as `/orders/:orderId`; query parameters commonly represent filters, sorting, and pagination.
- Navigation should use router-aware links or navigation APIs so the history and rendering model stay coordinated.
- A route change can invalidate data, cancel an old request, or require a deliberate loading and error state.

## Application

Use route parameters for resource identity and query parameters for shareable view state. Keep navigation, data loading, and permission boundaries close to the route that owns them.

## Common Mistakes

- Storing shareable filters only in component state so refresh and back/forward lose them.
- Reading route parameters without handling missing, malformed, or unauthorized values.
- Using ordinary anchors for in-app navigation when router links are needed for SPA behavior.
- Keeping a stale request alive after the route changes.
- Building deeply nested route logic that duplicates layout and authorization decisions.

## Common Interview Questions

### Foundation

- What is client-side routing?
- When should a value be a path parameter versus a query parameter?

### Intermediate

- How do nested routes share layout while rendering different content?
- How should a route change interact with an in-flight data request?

### Advanced and Follow-up

- How would you preserve a search/filter state across refresh and back navigation?
- Where should route-level authorization and not-found handling live?

### Code Prediction

Given `/customers/42?tab=orders&page=2`, identify which values represent resource identity and which represent view state, and explain which should survive a refresh.

## Practical Tasks

- Design routes for a paginated, filterable ASP.NET Core resource.
- Add route-aware request cancellation so a previous record cannot overwrite the newly selected record.

## Readiness Criteria

You can explain route identity, nested layouts, URL state, navigation, and the relationship between route changes and data loading.

## References

- [React Router: Main concepts](https://reactrouter.com/home)
- [React Router: Nested routes](https://reactrouter.com/start/framework/routing)
- [MDN: URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams)
