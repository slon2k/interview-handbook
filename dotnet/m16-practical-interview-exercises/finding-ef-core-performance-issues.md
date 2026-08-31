# Finding EF Core Performance Issues

## What This Assesses

Given a piece of EF Core-backed code, can you spot the specific performance problem (N+1, unnecessary tracking, cartesian explosion) by reading the code alone — the same skill Module 10 built, now under interview pressure with no query-logging tool available to lean on.

## Format and Time Expectations

A short method (10-20 lines) using `DbContext`, with a prompt like "what's wrong with this, performance-wise?" — explain both the problem and the specific fix, not just "this could be optimized."

## Exercise 1: The Classic N+1

**Problem:** What's the performance issue here, assuming lazy loading is enabled?

```csharp
var orders = await context.Orders.ToListAsync();
foreach (var order in orders)
{
    Console.WriteLine(order.Customer.Name);
}
```

**What a strong answer demonstrates:** Immediately naming this as N+1 (Module 10) — one query for `orders`, then one additional query *per order* triggered by accessing `order.Customer` under lazy loading. Fix: `.Include(o => o.Customer)` on the original query, or a projecting `Select` if only `Customer.Name` is actually needed.

**Common mistakes:** Suggesting "add caching" as the fix, which doesn't address the actual root cause (redundant per-item queries) at all.

## Exercise 2: Unnecessary Tracking on a Read-Only Path

**Problem:** What's inefficient about this, given it's a read-only reporting endpoint?

```csharp
[HttpGet("report")]
public async Task<IActionResult> GetReport()
{
    var orders = await context.Orders.Where(o => o.OrderDate >= DateTime.UtcNow.AddDays(-30)).ToListAsync();
    return Ok(orders.Select(o => new { o.Id, o.Total }));
}
```

**What a strong answer demonstrates:** Recognizing that this query is tracked by default (Module 10) despite never being modified or saved, incurring unnecessary change-tracker memory overhead. Fix: `.AsNoTracking()` on the query, and ideally project directly in the query (`.Select(...)` before `ToListAsync()`) rather than materializing full entities and shaping them afterward.

**Common mistakes:** Only adding `.AsNoTracking()` without also noting the missed opportunity to project earlier and reduce the columns actually fetched from the database.

## Exercise 3: Cartesian Explosion from Multiple Includes

**Problem:** What's the performance risk here for an order with many items and many payments?

```csharp
var order = await context.Orders
    .Include(o => o.Items)
    .Include(o => o.Payments)
    .FirstAsync(o => o.Id == id);
```

**What a strong answer demonstrates:** Recognizing the cartesian-explosion risk (Module 10) — joining two collection navigations in one query multiplies the raw row count (items × payments) before EF Core de-duplicates it back into the object graph. Fix: `.AsSplitQuery()` to issue separate queries per included collection instead.

**Common mistakes:** Not recognizing this as a distinct problem from N+1 — it's the *opposite* failure mode (too much data in one query, rather than too many queries) and needs a different fix.

## Exercise 4: A Method That Looks Fine But Isn't

**Problem:** This endpoint returns entities directly for JSON serialization, with lazy loading enabled globally. What's the hidden risk, given there's no visible loop in this method at all?

```csharp
[HttpGet]
public async Task<IActionResult> GetOrders() => Ok(await context.Orders.ToListAsync());
```

**What a strong answer demonstrates:** Recognizing that N+1 can hide behind JSON serialization itself (Module 10) — if the serializer walks every navigation property on every order while producing the response, lazy loading fires a query per order per navigation property accessed, with no `foreach` loop visible anywhere in the application code.

**Common mistakes:** Concluding this method is safe simply because there's no explicit loop, missing that the serializer's own traversal can trigger the exact same problem.

## Readiness Criteria

Identify N+1, unnecessary tracking, and cartesian-explosion patterns from reading code alone, propose the specific correct fix for each (not a generic "add caching"), and recognize that N+1 can hide behind serialization as well as an explicit loop.

## References

- [N+1 queries (Module 10)](../m10-entity-framework-core/n-plus-one-queries.md)
- [AsNoTracking (Module 10)](../m10-entity-framework-core/asnotracking.md)
- [Split vs. single queries (Module 10)](../m10-entity-framework-core/split-vs-single-queries.md)
- [Database query performance and N+1 problems (Module 13)](../m13-performance-diagnostics-observability/database-query-performance-and-n-plus-one.md)
