# Caching — the System-Design View

## Definition

This is the third and final angle on caching in this handbook: Module 8 covered `IMemoryCache`/Output Caching mechanics, Module 13 covered caching strategy (cache-aside, invalidation, in-memory vs. distributed decision-making). Here, the question is purely topological: in a multi-service system, *where* does the cache physically live, and how does that placement affect consistency and coupling between services.

```
Option A: each service has its OWN local cache (in-memory or its own Redis instance)
Option B: services SHARE one centralized distributed cache
Option C: a caching layer sits in FRONT of a service (a CDN, an API gateway cache) rather than inside it
```

## Alternatives & Trade-offs

A per-service local cache keeps each service simple and independent, but means the same data might be cached inconsistently across services if more than one needs the same underlying information — and it's exactly the Module 8 `IMemoryCache` cross-instance-inconsistency problem, now viewed across service boundaries rather than just scaled-out instances of one service. A shared distributed cache gives one consistent view across every consumer, at the cost of introducing a shared dependency that couples otherwise-independent services to the same infrastructure and invalidation logic. A front-of-service cache (CDN, gateway) is excellent for content that's cacheable independent of any specific backend service's internal state, but doesn't help with service-internal data needs at all.

## How It Works

### Per-service local cache — simple, but potentially inconsistent across services

```
Order Service caches Customer data locally (its own Redis instance or in-memory cache).
Shipping Service ALSO caches the same Customer data, independently, with its own TTL.
If Customer data changes, both caches must be invalidated separately — and if only one is,
the two services now have inconsistent views of the same customer for a while.
```

### Shared distributed cache — one consistent view, at the cost of a shared dependency

```
Order Service and Shipping Service both read/write to the SAME Redis instance for Customer data.
A single invalidation on a customer update is visible to both services immediately.
But: both services now depend on this shared Redis instance's availability, and changing its
schema/keying scheme requires coordinating across every service that shares it.
```

This mirrors the classic monolith-vs-microservices coupling trade-off (a later topic): centralizing the cache reduces inconsistency risk but reintroduces a shared dependency exactly the kind of thing splitting into services was meant to avoid.

### Front-of-service caching — CDN and API gateway layers

```
A CDN caches static or rarely-changing, publicly cacheable responses (Module 7's Cache-Control
headers) close to the end user, entirely outside any individual service's awareness.
An API gateway cache can cache full responses for common, cacheable requests before they even
reach the backend services at all — reducing load on every downstream service uniformly.
```

This layer doesn't solve service-internal caching needs, but is often the highest-leverage caching investment for read-heavy, publicly-facing traffic, since it protects every backend service simultaneously without each needing its own caching logic.

## Application

Default to per-service local/distributed caching (Module 13's territory) for data a single service needs for its own internal performance. Introduce a shared distributed cache specifically when multiple services need a genuinely consistent view of the same cached data and can tolerate the resulting coupling. Add a front-of-service cache (CDN, gateway) for publicly-facing, broadly cacheable content, independent of any individual service's internal caching strategy.

## Common Mistakes

- Each service independently caching the same underlying data with no coordination, creating cross-service inconsistency that's hard to notice until it causes a visible bug.
- Introducing a shared distributed cache as a default without weighing the coupling cost it reintroduces across otherwise-independent services.
- Not considering a front-of-service cache (CDN/gateway) for publicly cacheable content, missing the highest-leverage caching opportunity for read-heavy public traffic.
- Confusing this topological question with the strategic question from Module 13 (cache-aside vs. read-through, invalidation patterns) — they're complementary decisions, not the same one.

## Common Interview Questions

### Basic
- What are the three general places a cache can live in a multi-service system?
- What's the risk of each service independently caching the same underlying data?

### Intermediate
- What coupling cost does a shared distributed cache reintroduce across otherwise-independent services?
- What kind of traffic benefits most from a front-of-service (CDN/gateway) cache?

### Advanced
- How would you decide, for a specific piece of data needed by multiple services, whether to cache it locally per-service or in a shared distributed cache?
- How does this topological caching decision interact with the modular-monolith-vs-microservices trade-off covered elsewhere in this module?

### Follow-up Questions
- Does a front-of-service cache eliminate the need for any service-internal caching?
- Can a system use more than one of these three caching placements simultaneously?

### Code Prediction
Two services each maintain their own local cache of the same `Customer` entity, with independent 10-minute expirations and no coordinated invalidation. A customer's address is updated via one service. For how long could the two services disagree about that customer's address, in the worst case?

## Practical Tasks

- Design a caching topology for a multi-service system with both service-internal caching needs and publicly-facing, broadly cacheable content.
- Identify a scenario where a shared distributed cache's consistency benefit is worth its coupling cost, and one where it isn't.
- Design coordinated invalidation for data cached independently by two services sharing the same underlying source of truth.

## Readiness Criteria

Choose an appropriate caching topology (per-service, shared, or front-of-service) for a given multi-service scenario, and reason about the consistency-versus-coupling trade-off each placement makes.

## References

### Other

- [Distributed caching strategies (Module 13)](../m13-performance-diagnostics-observability/caching-strategies.md)
