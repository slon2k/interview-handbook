# LINQ Tasks

## What This Assesses

LINQ fluency specifically — not just "can you solve this problem," but "can you express it idiomatically using LINQ operators rather than falling back to manual loops." Interviewers are also watching whether you understand what each operator actually does (Module 3's deferred execution, complexity) rather than just pattern-matching syntax you've memorized.

## Format and Time Expectations

Usually a short data-transformation prompt against a described or given collection of objects, solved live, often followed by "now what if [additional requirement]" to see how you adapt the query rather than rewriting from scratch.

## Exercise 1: Group and Aggregate

**Problem:** Given a list of `Order { int CustomerId, decimal Total, DateTime OrderDate }`, return each customer's total spend and order count, for customers with more than 2 orders.

```csharp
var result = orders
    .GroupBy(o => o.CustomerId)
    .Where(g => g.Count() > 2)
    .Select(g => new { CustomerId = g.Key, TotalSpend = g.Sum(o => o.Total), OrderCount = g.Count() });
```

**What a strong answer demonstrates:** Correct `GroupBy` → `Where` → `Select` ordering (filtering groups *after* grouping, mirroring Module 9's `HAVING` vs. `WHERE` distinction at the SQL level); avoiding calling `g.Count()` and `g.Where(...).Count()` redundantly when one suffices.

**Common mistakes:** Filtering with `.Where(o => ...)` on individual orders before grouping when the actual filter condition is about the *group* (order count), producing a completely different, incorrect result.

## Exercise 2: Top N Per Category

**Problem:** Given the same orders, return each customer's 3 most recent orders.

```csharp
var result = orders
    .GroupBy(o => o.CustomerId)
    .Select(g => new { CustomerId = g.Key, RecentOrders = g.OrderByDescending(o => o.OrderDate).Take(3) });
```

**What a strong answer demonstrates:** Recognizing this as "top N per group" (Module 9's window-function equivalent, `ROW_NUMBER() OVER (PARTITION BY ...)`) and correctly ordering *within* each group before taking, not ordering the flat list globally first.

**Common mistakes:** Calling `.OrderByDescending(...).Take(3)` on the whole ungrouped collection first, which returns only 3 orders total across *all* customers, not 3 per customer.

## Exercise 3: Deferred Execution Trap

**Problem:** Explain what this code prints, and why.

```csharp
var numbers = new List<int> { 1, 2, 3 };
var query = numbers.Select(n => n * 2);
numbers.Add(4);
foreach (var n in query) Console.WriteLine(n);
```

**What a strong answer demonstrates:** Correctly identifying that `query` is not evaluated until enumerated in the `foreach` — by which point `numbers` already contains 4 — so the output is `2, 4, 6, 8`, not `2, 4, 6` (Module 3's deferred-execution content, applied precisely rather than just recited).

**Common mistakes:** Assuming `Select` executes immediately at the point it's written, missing that the later mutation to `numbers` is actually visible in the final enumeration.

## Exercise 4: Flattening a Nested Collection

**Problem:** Given a list of `Order`, each with a list of `OrderItem`, return a flat list of all item SKUs across every order.

```csharp
var skus = orders.SelectMany(o => o.Items).Select(i => i.Sku).ToList();
```

**What a strong answer demonstrates:** Reaching for `SelectMany` directly instead of a nested loop or a `Select` producing a collection-of-collections that then needs manual flattening.

**Common mistakes:** Using `Select(o => o.Items.Select(i => i.Sku))`, which produces `IEnumerable<IEnumerable<string>>` — a jagged result — instead of the flat `IEnumerable<string>` the problem actually asked for.

## Readiness Criteria

Write each of these fluently without needing to look up operator names, correctly predict deferred-execution behavior, and recognize "top N per group" and "flatten a nested collection" as named patterns rather than solving them from first principles each time.

## References

- [LINQ query operators (Module 3)](../m03-collections-linq/linq-query-operators.md)
- [Deferred execution and materialization (Module 3)](../m03-collections-linq/deferred-execution-and-materialization.md)
- [Window functions (Module 9)](../m09-relational-databases-and-sql/window-functions.md)
