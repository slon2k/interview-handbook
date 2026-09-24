# Bundling, Tree Shaking, Lazy Loading, and Code Splitting

## Definition

A bundler builds a module graph into browser assets. Tree shaking removes code proven unused by static imports and side-effect information. Dynamic imports create split points so code can load only when a feature needs it.

## Alternatives & Trade-offs

One bundle simplifies requests but delays initial load as an application grows. Splitting reduces initial JavaScript but adds network boundaries, loading states, and possible waterfalls. Split by meaningful routes or expensive optional features, not every small component.

## How It Works

Static ESM imports let tools analyze usage. Dynamic `import()` produces a separately loadable chunk. Tree shaking is limited by side effects, CommonJS interop, opaque barrel imports, and packages that inaccurately declare side-effect behavior. React `lazy` and `Suspense` mechanics belong in Module 5; this topic explains their delivery cost.

## Application

Measure the initial route before splitting. Code-split an admin area, rich editor, charting feature, or rarely opened dialog when data shows it is costly. Provide a purposeful loading and failure state around each split boundary.

## Common Mistakes

- Assuming every unused export is removed regardless of module format and side effects.
- Creating many tiny chunks that serialize requests.
- Adding lazy loading without a loading state or error boundary.
- Optimizing bundles without first inspecting their contents.

## Common Interview Questions

- What conditions make tree shaking effective?
- What is the trade-off of code splitting?
- Why is route-level splitting often a useful default?

## Practical Tasks

- Convert an expensive optional feature to a dynamic import and define its loading state.
- Inspect a dependency that prevents tree shaking and explain its effect on the module graph.

## Readiness Criteria

You can explain the relationship between imports, chunks, and initial load, and choose split points based on measured user value.

## References

- [Vite build optimization](https://vite.dev/guide/features#async-chunk-loading-optimization)
- [MDN dynamic import](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/import)
