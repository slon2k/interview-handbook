# Browser Caching and Frontend Delivery

## Definition

Frontend delivery publishes HTML, JavaScript, CSS, images, and other assets to a browser-facing host or CDN. Browser caching stores reusable responses under HTTP cache rules. Content-hashed assets can be cached for a long time because a changed file receives a different URL.

## Alternatives & Trade-offs

Long-lived immutable caching improves repeat visits but requires correct content hashing and HTML delivery. Short-lived assets reduce stale-content risk but sacrifice cache value. HTML commonly needs fresher validation because it points at the current hashed assets; versioned assets can use much longer cache lifetimes.

## How It Works

A production build emits assets with content-based names. A deployment publishes them before or atomically with the HTML that references them. Cache headers guide reuse and revalidation. A rollback must account for clients that already have old HTML or assets, APIs that changed compatibility, and CDN propagation.

## Application

Use immutable caching for hashed static assets, a deliberate policy for HTML, and a CDN where it fits the delivery model. Keep frontend API changes backward-compatible during deployment transitions when clients may run more than one frontend version.

## Common Mistakes

- Caching HTML indefinitely while it references changed entry assets.
- Deploying HTML before referenced assets are available.
- Assuming a hard refresh is a production cache strategy.
- Releasing a breaking API change while cached clients still use the previous contract.

## Common Interview Questions

- Why can hashed assets be cached longer than HTML?
- What deployment ordering avoids broken asset references?
- How does caching affect frontend-backend compatibility?

## Practical Tasks

- Inspect a production build's hashed assets and propose cache headers for HTML versus versioned files.
- Describe a rollback plan that accounts for cached frontend versions.

## Readiness Criteria

You can explain the relationship between hashed assets, cache headers, deployment order, and frontend/API compatibility.

## References

- [MDN HTTP caching](https://developer.mozilla.org/docs/Web/HTTP/Guides/Caching)
- [web.dev cache HTTP](https://web.dev/articles/http-cache)
