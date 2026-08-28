# Caching Strategies: In-Memory vs. Distributed

## Definition

Module 8 covered `IMemoryCache` and Output Caching mechanics; this topic covers the *strategic* decisions sitting above those mechanics: which caching pattern to use (cache-aside, read-through, write-through), how to invalidate correctly, and when in-memory caching's per-instance limitation (also from Module 8) actually forces a move to a distributed cache like Redis.

```
Cache-aside:   application code checks the cache first, falls back to the source on a miss,
               and populates the cache — the most common pattern, full control in application code.
Read-through:  the cache itself knows how to load from the source on a miss — the caching layer
               is smarter, application code just asks the cache.
Write-through: writes go through the cache, which writes to the source synchronously — keeps
               cache and source always consistent, at the cost of write latency.
```

## Alternatives & Trade-offs

Cache-aside is the most common pattern because it requires no special cache-layer support — any cache (in-memory or distributed) can be used this way, with the application fully controlling load and invalidation logic. Read-through/write-through shift some of that logic into the cache layer itself, which can simplify application code but requires a cache implementation that supports it. In-memory caching is fast (no network hop) but inconsistent across scaled-out instances (Module 8); distributed caching (Redis, etc.) gives consistency across instances at the cost of network latency per cache access.

## How It Works

### Cache-aside — the default pattern

```csharp
public async Task<Product> GetProductAsync(int id)
{
    if (_cache.TryGetValue($"product:{id}", out Product cached)) return cached;

    var product = await _repository.GetByIdAsync(id); // fall back to the source of truth
    _cache.Set($"product:{id}", product, TimeSpan.FromMinutes(10));
    return product;
}
```

### Invalidation — the hardest part of any caching strategy

```csharp
public async Task UpdateProductAsync(Product product)
{
    await _repository.UpdateAsync(product);
    _cache.Remove($"product:{product.Id}"); // explicit invalidation on write — easy to forget at every write path
}
```

Every code path that changes data the cache holds needs to remember to invalidate it — a distributed system with several services able to write the same underlying data multiplies this risk, since invalidation logic living only in one service's cache-aside code doesn't help another service's stale cached copy.

### Moving from in-memory to distributed caching — the trigger, revisited from Module 8

```
In-memory (IMemoryCache): fine for a single instance, or for data where slight staleness/inconsistency
                           across instances is acceptable.
Distributed (Redis, etc.): needed once multiple instances must see a consistent, shared cache state —
                           e.g., invalidating a cache entry on one instance must be visible to all others
                           immediately, which IMemoryCache structurally cannot provide (Module 8).
```

```csharp
builder.Services.AddStackExchangeRedisCache(options => options.Configuration = "redis-connection-string");
```

### Cache stampede — a subtle failure mode under high concurrency

```csharp
// If a popular cache key expires and 1,000 concurrent requests all miss at once,
// all 1,000 might hit the database simultaneously trying to repopulate the same cache entry
```

A common mitigation is locking around the cache-population step (so only one request repopulates while others wait briefly for the result) or staggering expiration times to avoid many popular keys expiring simultaneously.

## Application

Use cache-aside as the default pattern unless a specific cache technology's read-through/write-through support genuinely simplifies things. Move from in-memory to distributed caching specifically when cross-instance consistency matters, not by default. Design explicit invalidation for every write path that changes cached data, and consider stampede protection for high-traffic cache keys.

## Common Mistakes

- Choosing in-memory caching for data that needs cross-instance consistency, not realizing the limitation until stale data is observed inconsistently across requests hitting different instances.
- Forgetting to invalidate a cache entry on one of several write paths that modify the same underlying data, leaving it stale indefinitely until natural expiration.
- Not considering cache stampede risk for a very popular cache key with a hard expiration, causing a burst of simultaneous database load when it expires under high traffic.
- Treating caching as a default performance fix applied everywhere, rather than a deliberate choice justified by measured read-heavy access patterns (tying back to "measure before optimizing").

## Common Interview Questions

### Basic
- What's the difference between cache-aside, read-through, and write-through caching?
- When would you choose a distributed cache over an in-memory one?

### Intermediate
- Why is cache invalidation often described as "the hard part" of caching?
- What is a cache stampede, and how would you mitigate it?

### Advanced
- How would you design a caching strategy for data written by multiple services, ensuring invalidation is reliable across all of them?
- How would you decide, for a specific dataset, whether the staleness risk of in-memory caching is acceptable versus requiring a distributed cache?

### Follow-up Questions
- Does a distributed cache eliminate the need for careful invalidation logic?
- Can cache-aside be used with a distributed cache, or is it specific to in-memory caching?

### Code Prediction
An application scaled to three instances uses in-memory caching with cache-aside, and only invalidates the cache on the specific instance that handles an update request. What do the other two instances' cached copies look like immediately after the update, and for how long does that inconsistency persist?

## Practical Tasks

- Implement cache-aside for a read-heavy endpoint, including explicit invalidation on the corresponding write path.
- Migrate an in-memory caching implementation to a distributed cache (Redis) and verify cross-instance consistency.
- Design a stampede-mitigation strategy for a specific high-traffic cache key with a hard expiration.

## Readiness Criteria

Choose the correct caching pattern and cache technology (in-memory vs. distributed) based on actual consistency requirements, design reliable invalidation across every relevant write path, and mitigate cache stampede for high-traffic keys.

## References

### Microsoft Learn

- [Cache in-memory in ASP.NET Core](https://learn.microsoft.com/aspnet/core/performance/caching/memory)
- [Distributed caching in ASP.NET Core](https://learn.microsoft.com/aspnet/core/performance/caching/distributed)
