# Core Web Vitals, Network Waterfalls, and Render Profiling

## Definition

Core Web Vitals are user-centric metrics: Largest Contentful Paint (LCP) reflects loading, Interaction to Next Paint (INP) reflects interaction responsiveness, and Cumulative Layout Shift (CLS) reflects visual stability. Network waterfalls and browser render profiles help identify the resources and work causing a measured problem.

## Alternatives & Trade-offs

Lab measurements are repeatable and useful during development; field measurements represent real users and devices but are noisier. Both matter. Do not optimize from a metric label alone: identify the resource, CPU task, render, or layout behavior that produces it.

## How It Works

Use browser DevTools to inspect request priority, blocking resources, long tasks, layout shifts, and render work. Record a profile around a user action, then connect the dominant cost to a change. React profiling can identify avoidable component work, but it does not replace browser-level measurement.

## Application

Measure a slow initial page or interaction, form a falsifiable hypothesis, change one contributor, and measure again. Keep general service-side telemetry and backend diagnostics in the .NET performance module; this topic owns the browser-facing evidence.

## Common Mistakes

- Treating a synthetic Lighthouse score as the complete user experience.
- Memoizing React components before finding a measurable rendering bottleneck.
- Optimizing a network waterfall without checking CPU and rendering work.
- Removing image dimensions and causing layout shifts.

## Common Interview Questions

- What do LCP, INP, and CLS broadly measure?
- How would you investigate a slow interaction?
- Why measure before and after an optimization?

## Practical Tasks

- Record a performance trace for a slow page and state one testable cause.
- Measure an interaction before and after a targeted change, then report the evidence rather than an assumption.

## Readiness Criteria

You can interpret browser performance evidence, differentiate loading, interaction, and stability problems, and validate an optimization with measurement.

## References

- [web.dev: Core Web Vitals](https://web.dev/articles/vitals)
- [Chrome DevTools performance analysis](https://developer.chrome.com/docs/devtools/performance)
