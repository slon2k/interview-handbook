# Client-Side Routing, URLs, and History State

## Definition

Client-side routing lets a single-page app update the URL and render different views without a full page reload. The browser's history API enables navigation and state management while keeping the app responsive.

## How It Works

- A route maps a URL path or query string to a view or component.
- Browser navigation changes the URL and the app decides what to render based on that route.
- `history.pushState()` and `replaceState()` update the browser history without reloading the page.
- Query parameters and hash fragments often represent UI state that is shareable or bookmarkable.
- Route state can be local UI state, server state, or URL state; the right choice depends on whether it should persist across reloads or share across sessions.

## Application

Use URL state for navigation and shareable context, and keep ephemeral UI state in local component state. This makes the app more predictable and easier to debug than storing too much in ad hoc state or component memory.

## Common Mistakes

- Treating every route parameter as local state instead of a navigable source of truth.
- Using non-serializable or transient state in the URL and then wondering why the page is inconsistent after refresh.
- Forgetting that browser history is stateful and that `back` and `forward` are user-level navigation events.
- Mixing local UI state with URL state and creating hidden coupling between components.

## Common Interview Questions

### Foundation

- Why does client-side routing require a router library or a custom route map?
- What is the difference between URL state and local component state?

### Intermediate

- How would you preserve a filter state across navigation and refresh?
- When should a route parameter be used instead of a query string?

### Advanced and Follow-up

- How do you handle navigation when the route changes and a request is already in flight?
- Why might `pushState` be a better fit than full page navigation for a modal or filter workflow?

### Code Prediction

Given a URL like `/orders?status=open&priority=high`, explain what belongs in route state versus component state and how a refresh should restore the list view.

## Practical Tasks

- Build a small router that maps a path and query string to a view.
- Explain when a filter should be represented in the URL versus component state.

## Readiness Criteria

You can explain URL state, history management, routing, and the difference between shareable application state and local UI state.

## References

- [MDN: History API](https://developer.mozilla.org/en-US/docs/Web/API/History)
- [MDN: URL and URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URL)
- [MDN: Location](https://developer.mozilla.org/en-US/docs/Web/API/Location)
