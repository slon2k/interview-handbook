# Bundle Analysis and Asset Optimization

## Definition

Bundle analysis identifies which modules and assets contribute most to downloaded and executed page cost. Asset optimization chooses appropriate image dimensions, formats, compression, and loading behavior so media supports rather than dominates the user experience.

## Alternatives & Trade-offs

Replacing a large dependency can reduce JavaScript cost but adds migration risk. Aggressive image compression or lazy loading can reduce bytes but harm visual quality or delay critical content. Optimize the critical route first and use a performance budget to make trade-offs visible.

## How It Works

Inspect emitted chunks and source maps with a bundle visualizer. Distinguish transfer size from parsed and executed JavaScript cost. For images, serve dimensions close to the rendered size, responsive variants where appropriate, modern formats supported by the delivery path, and explicit width/height to reduce layout shift.

## Application

Analyze builds after adding a UI library, editor, charting package, or large media. Lazy-load below-the-fold media where it does not delay a user task. Preload only assets that are genuinely critical; unnecessary preloads compete with more important resources.

## Common Mistakes

- Optimizing a small asset while a large dependency dominates script cost.
- Shipping original camera images at thumbnail dimensions.
- Lazy-loading the main visual content that determines LCP.
- Measuring only gzip size and ignoring execution cost.

## Common Interview Questions

- What would you inspect after a bundle-size regression?
- Why should images declare dimensions?
- When can lazy loading make perceived performance worse?

## Practical Tasks

- Produce a bundle report and identify the three largest contributors to an initial route.
- Optimize a responsive image while preserving its layout dimensions and critical-loading behavior.

## Readiness Criteria

You can use a bundle report to form a performance hypothesis, prioritize the dominant cost, and optimize assets without creating regressions.

## References

- [web.dev: optimize images](https://web.dev/learn/performance/image-performance)
- [Vite static asset handling](https://vite.dev/guide/assets)
