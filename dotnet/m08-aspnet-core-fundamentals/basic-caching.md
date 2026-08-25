# Basic Caching (Framework Mechanics)

## Definition

ASP.NET Core provides in-process caching via `IMemoryCache`, and two HTTP-response-level caching mechanisms: **Response Caching** middleware (respects/sets `Cache-Control` headers, can cache at a reverse proxy or client) and **Output Caching** (.NET 7+, caches the server-generated response itself, including for dynamic endpoints, without requiring the client or proxy to cooperate).

```csharp
builder.Services.AddMemoryCache();

public class ProductService
{
    private readonly IMemoryCache _cache;
    public ProductService(IMemoryCache cache) => _cache = cache;

    public async Task<Product> GetAsync(int id) =>
        await _cache.GetOrCreateAsync($"product:{id}", async entry =>
        {
            entry.SlidingExpiration = TimeSpan.FromMinutes(10);
            return await LoadFromDatabaseAsync(id);
        });
}
```

## Alternatives & Trade-offs

`IMemoryCache` is simple and fast (in-process, no network hop) but doesn't scale across multiple instances — each instance has its own independent cache, so a cache invalidation on one instance doesn't affect the others. Output Caching works at the HTTP response level and can be backed by a distributed store, giving consistent caching across scaled-out instances, but caches whole responses rather than arbitrary application data. Distributed caching (Redis, etc. — covered in Module 13 - Performance) solves the multi-instance consistency problem for arbitrary cached data, at the cost of a network round-trip per cache access instead of in-process memory access.

## How It Works

### `IMemoryCache` — in-process, per-instance

```csharp
_cache.Set("key", value, TimeSpan.FromMinutes(5)); // absolute expiration
_cache.GetOrCreateAsync("key", async entry =>
{
    entry.SlidingExpiration = TimeSpan.FromMinutes(2); // resets on each access, up to an absolute cap if also set
    return await LoadExpensiveDataAsync();
});
```

If the app runs as three scaled-out instances, each has its own separate `IMemoryCache` — invalidating a key on instance A has no effect on instances B and C, which is fine for read-mostly, eventually-consistent data but wrong for anything requiring immediate cross-instance consistency.

### Output Caching (.NET 7+) — caches the whole generated response

```csharp
builder.Services.AddOutputCache(options =>
{
    options.AddPolicy("ProductsPolicy", policy => policy.Expire(TimeSpan.FromMinutes(5)).Tag("products"));
});

app.UseOutputCache();
app.MapGet("/products/{id:int}", GetProduct).CacheOutput("ProductsPolicy");
```

```csharp
// Invalidate the cached response(s) tagged "products" whenever a product changes
app.MapPost("/products", async (Product product, IOutputCacheStore cacheStore) =>
{
    await SaveProductAsync(product);
    await cacheStore.EvictByTagAsync("products", default); // invalidates cached GET responses tagged "products"
    return Results.Created();
});
```

### Response Caching — respects HTTP caching semantics

```csharp
app.UseResponseCaching();
app.MapGet("/products/{id:int}", (int id) => Results.Ok(GetProduct(id)))
   .CacheOutput()  // or set Cache-Control headers manually for Response Caching middleware
;
```

Response Caching works with standard `Cache-Control`/`ETag` headers (see Module 7's `headers-and-content-types.md`), so it can be honored by intermediate proxies and browsers too, not just the server itself.

## Application

Use `IMemoryCache` for small, per-instance, read-mostly data where slight inconsistency across scaled-out instances is acceptable (reference/lookup data, expensive computed values). Use Output Caching for whole-response caching of dynamic GET endpoints with tag-based invalidation. Reach for distributed caching (Module 13) once cross-instance consistency genuinely matters.

## Common Mistakes

- Using `IMemoryCache` for data that must be consistent across scaled-out instances, not realizing each instance caches independently.
- Caching a GET endpoint's output without a clear invalidation strategy, serving stale data indefinitely after the underlying data changes.
- Setting an unbounded or excessively long cache duration for data that changes more frequently than assumed.
- Confusing Response Caching (respects client/proxy-visible HTTP headers) with Output Caching (server-side, works even for endpoints that wouldn't normally be cacheable via headers alone) and picking the wrong one for the scenario.

## Common Interview Questions

### Basic
- What is `IMemoryCache`, and what's its main limitation in a scaled-out deployment?
- What's the difference between Response Caching and Output Caching?

### Intermediate
- How would you invalidate a cached GET endpoint's response when the underlying data changes?
- Why doesn't `IMemoryCache` provide consistency across multiple scaled-out instances?

### Advanced
- How would you design a tag-based invalidation strategy for Output Caching across several related endpoints?
- When would you migrate from `IMemoryCache` to a distributed cache, and what would break if you didn't?

### Follow-up Questions
- Does Output Caching require the client to send any specific headers to benefit from it?
- Can `IMemoryCache` entries be evicted before their expiration time?

### Code Prediction
An app scaled to three instances uses `IMemoryCache` to cache product data with a 10-minute sliding expiration. A product is updated via an admin panel and the code explicitly calls `_cache.Remove("product:42")` on the instance that handled the update request. What happens to the other two instances' cached copies of that product?

## Practical Tasks

- Implement `IMemoryCache`-based caching for an expensive read operation with appropriate expiration.
- Implement Output Caching with tag-based invalidation for a set of related GET endpoints.
- Explain, for a given scaled-out deployment, why `IMemoryCache` alone is insufficient and what alternative would fix it.

## Readiness Criteria

Explain the scope and limitations of `IMemoryCache` in a scaled-out deployment, implement Output Caching with tag-based invalidation, and know when to escalate to distributed caching.

## References

### Microsoft Learn

- [Cache in-memory in ASP.NET Core](https://learn.microsoft.com/aspnet/core/performance/caching/memory)
- [Output caching middleware](https://learn.microsoft.com/aspnet/core/performance/caching/output)
- [Response caching middleware](https://learn.microsoft.com/aspnet/core/performance/caching/middleware)
