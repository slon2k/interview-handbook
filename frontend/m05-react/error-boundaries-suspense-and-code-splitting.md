# Error Boundaries, Suspense, and Code Splitting

## Definition

An **error boundary** catches rendering errors thrown by its descendant component tree and renders a fallback instead of crashing the whole application. **Suspense** shows fallback UI while something a component depends on isn't ready yet — most commonly, a lazily-loaded component's code still downloading. **Code splitting** (`React.lazy`) defers loading part of the application's JavaScript until it's actually needed.

```tsx
<ErrorBoundary fallback={<ErrorMessage />}>
  <Suspense fallback={<Spinner />}>
    <LazyDashboard />
  </Suspense>
</ErrorBoundary>
```

## Alternatives & Trade-offs

Loading the entire application's JavaScript upfront is simpler — no `Suspense` boundaries to design, no loading states for code itself — but means every user pays the download cost of every feature, even ones they never visit. Code splitting defers that cost to the point of actual need, at the cost of needing a `Suspense` fallback (a loading state for *code*, not data) and careful thought about where the natural split points are — usually routes or clearly separate features.

## How It Works

### A `try`/`catch` around JSX does not catch rendering errors — a real error boundary is required

```tsx
function BrokenAttempt() {
  try {
    return <ComponentThatThrows />; // a render-time exception here is NOT caught by this try/catch
  } catch (error) {
    return <ErrorMessage />; // this line never actually runs for a rendering error
  }
}
```

```tsx
// A real error boundary: implemented as a class component, since there's no hook-based equivalent
class ErrorBoundary extends React.Component<
  { fallback: ReactNode; children: ReactNode },
  { hasError: boolean }
> {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true }; // triggered when a DESCENDANT throws during rendering
  }

  render() {
    if (this.state.hasError) return this.props.fallback;
    return this.props.children;
  }
}
```

`try`/`catch` only catches exceptions thrown synchronously within the exact block it wraps — it has no way to intercept an error thrown deep inside a child component's own render function. Error boundaries exist specifically because React needs a dedicated mechanism (`getDerivedStateFromError`) to catch failures from anywhere in a descendant subtree.

### An error boundary catches render errors — not event handler errors, not async errors

```tsx
function Form() {
  function handleSubmit() {
    throw new Error("Validation failed"); // an error boundary does NOT catch this — it's inside an event handler, not rendering
  }
  return <button onClick={handleSubmit}>Submit</button>;
}
```

An error thrown inside an event handler, a `useEffect`, or a rejected promise is a completely separate concern from a rendering error — none of those are caught by an error boundary, and each needs its own explicit handling (a `try`/`catch` in the handler, an error state from the async operation).

### `React.lazy` + `Suspense` — deferring a component's code until it's needed

```tsx
const Dashboard = React.lazy(() => import("./Dashboard")); // the Dashboard module isn't downloaded until rendered

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Dashboard /> {/* while Dashboard's code is still downloading, Spinner renders instead */}
    </Suspense>
  );
}
```

Without the surrounding `Suspense`, rendering a lazy component that hasn't finished loading its code has no defined fallback to show — `Suspense` is what tells React what to display during that gap.

### Route-based code splitting — the most common, highest-value split point

```tsx
const Dashboard = React.lazy(() => import("./routes/Dashboard"));
const Settings = React.lazy(() => import("./routes/Settings"));

<Routes>
  <Route path="/dashboard" element={<Suspense fallback={<PageSkeleton />}><Dashboard /></Suspense>} />
  <Route path="/settings" element={<Suspense fallback={<PageSkeleton />}><Settings /></Suspense>} />
</Routes>
```

A user visiting `/dashboard` never downloads `Settings`'s code at all until (and unless) they actually navigate there — splitting at route boundaries is usually the single highest-value place to introduce code splitting, since an entire route's worth of code is a natural, meaningful chunk to defer.

## Application

Place error boundaries around meaningful, independent feature or route boundaries so one broken subtree doesn't take down the whole page — and design each fallback to preserve orientation (a route shell with a placeholder) rather than replacing everything with a blank error message. Use `React.lazy` + `Suspense` at route or major-feature boundaries, where the deferred code is substantial enough to be worth the added loading state.

## Common Mistakes

- Trying to catch a rendering error with a plain `try`/`catch` around JSX instead of an actual error boundary.
- Expecting an error boundary to catch an error thrown inside an event handler or an async callback, when it only catches rendering-phase errors.
- Wrapping the entire application in exactly one top-level error boundary, so any single feature's failure takes down everything else too.
- Splitting very small components into their own lazy-loaded chunks, adding request overhead without any meaningful bundle-size benefit.
- Forgetting a `Suspense` boundary around a lazily-loaded component entirely, leaving no defined fallback while its code downloads.

## Common Interview Questions

### Basic
- What does an error boundary catch, and what does it not catch?
- What is `React.lazy` used for, and why does it need a `Suspense` boundary?

### Intermediate
- Why doesn't a normal `try`/`catch` around JSX catch a child component's rendering error?
- Why is route-based code splitting usually the highest-value place to start?

### Advanced
- Walk through why an error boundary catches a rendering-phase throw but not an error thrown inside a `useEffect` or an event handler.
- How would you design fallback UI that preserves the user's sense of place, rather than replacing the whole page with a spinner or blank error?

### Follow-up Questions
- Can a single error boundary usefully wrap the entire application, or should there typically be several?
- Does `Suspense` work only with `React.lazy`, or can it be used for other kinds of "not ready yet" state?

### Code Prediction
```tsx
function Page() {
  function handleClick() { throw new Error("boom"); }
  return (
    <ErrorBoundary fallback={<div>Something went wrong</div>}>
      <button onClick={handleClick}>Click me</button>
    </ErrorBoundary>
  );
}
```
If the button is clicked, does the error boundary's fallback appear? Why or why not, given where the `throw` actually happens?

## Practical Tasks

- Implement a class-based error boundary and verify it catches a deliberately thrown rendering error but not an error thrown from an event handler.
- Add route-based code splitting to a small multi-route app using `React.lazy` and `Suspense`, and verify each route's code loads only when visited.
- Design a fallback UI for a feature-level error boundary that preserves page layout instead of showing a blank message.

## Readiness Criteria

Explain precisely what an error boundary does and doesn't catch, implement one correctly, and use `React.lazy`/`Suspense` at meaningful split points with well-designed fallback UI.

## References

- [React: Catching Rendering Errors with an Error Boundary](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)
- [React: `lazy`](https://react.dev/reference/react/lazy)
- [React: `Suspense`](https://react.dev/reference/react/Suspense)
