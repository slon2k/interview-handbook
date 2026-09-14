# `useMemo`, `useCallback`, and Rendering Performance

## Definition

`useMemo` caches a calculated value and `useCallback` caches a function reference between renders when dependencies remain equal. Both are performance tools, not correctness tools.

## How It Works

- A memoized calculation is reused only when its dependency values compare equal.
- `useCallback(fn, dependencies)` is conceptually a way to preserve a function identity for consumers that care about reference equality.
- Memoization can help a child wrapped in `React.memo` or avoid repeating an expensive calculation, but it also adds dependency and maintenance cost.
- A memoized value can become stale when dependencies are incomplete.
- Profiling should identify the expensive render or calculation before optimization is added.

## Application

Start with correct state boundaries and simple rendering. Add memoization when a measured bottleneck or a deliberate referential-equality contract justifies it, then verify the result with profiling.

## Common Mistakes

- Adding `useMemo` to every calculation regardless of cost.
- Using `useCallback` without a memoized consumer or another identity-sensitive dependency.
- Omitting dependencies and freezing a stale value or function.
- Assuming memoization prevents all child renders.
- Optimizing before checking whether the real issue is excessive state scope or an unstable key.

## Common Interview Questions

### Foundation

- What do `useMemo` and `useCallback` do?
- Are they required for correctness?

### Intermediate

- When can `useCallback` help a memoized child?
- What costs does memoization introduce?

### Advanced and Follow-up

- How would you distinguish a real memoization opportunity from a state-ownership problem?
- Why can a memoized callback still be stale?

### Code Prediction

Given a memoized child and a callback whose dependency list omits `selectedId`, predict which value the callback uses after the selection changes.

## Practical Tasks

- Use the React Profiler to identify a costly calculation and justify whether `useMemo` is appropriate.
- Remove unnecessary memoization from a component and explain why the simpler version is safer.

## Readiness Criteria

You can explain referential equality, dependency correctness, profiling, and the trade-offs of `useMemo`, `useCallback`, and `React.memo`.

## References

- [React: `useMemo`](https://react.dev/reference/react/useMemo)
- [React: `useCallback`](https://react.dev/reference/react/useCallback)
- [React: React Developer Tools](https://react.dev/learn/react-developer-tools)
