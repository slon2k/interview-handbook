# Projection

## Definition

Projection shapes a query's result into something other than a full entity — a DTO, an anonymous type, or a subset of columns — using `Select()`. EF Core translates a projection into SQL that only fetches the needed columns, rather than the full entity's columns.

```csharp
var summaries = await context.Orders
    .Select(o => new OrderSummaryDto { Id = o.Id, Total = o.Total })
    .ToListAsync();
// generated SQL only selects Id and Total, not every column on Orders
```

## Alternatives & Trade-offs

Projecting to a DTO reduces the amount of data transferred from the database and avoids the overhead of change tracking for data that's only ever going to be read and serialized. Loading full entities is simpler when the data really will be modified and saved back, or when a DTO's extra type doesn't pull its weight — for a quick internal query, an anonymous type may be all that's needed instead of a named DTO class.

## How It Works

### Projection reduces fetched columns

```csharp
// Fetches every column on Orders, even though only two are actually used
var orders = await context.Orders.ToListAsync();
var summaries = orders.Select(o => new { o.Id, o.Total }).ToList(); // shaping happens in memory, too late

// Fetches only Id and Total directly from the database
var summaries = await context.Orders.Select(o => new { o.Id, o.Total }).ToListAsync();
```

The second version's `SELECT` in the generated SQL only includes `Id` and `Total` — the shaping happens as part of the database query itself, not as an afterthought in application memory.

### Projecting into related data without a full `Include`

```csharp
var summaries = await context.Orders
    .Select(o => new OrderSummaryDto
    {
        Id = o.Id,
        CustomerName = o.Customer.Name, // EF Core generates a JOIN, without needing an explicit Include()
        Total = o.Total
    })
    .ToListAsync();
```

Projection can pull in related data via a generated join automatically, often making an explicit `Include()` (see `loading-strategies.md`) unnecessary when only specific related fields are needed rather than the full related entity.

### Projected results are never tracked

```csharp
var summaries = await context.Orders.Select(o => new OrderSummaryDto { Id = o.Id }).ToListAsync();
// these DTOs are plain objects — there's no concept of "tracking" a projection result, regardless of AsNoTracking()
```

### Projecting a computed value

```csharp
var summaries = await context.Orders
    .Select(o => new { o.Id, IsLarge = o.Total > 1000 })
    .ToListAsync(); // the comparison is translated and evaluated in SQL, not pulled into memory first
```

## Application

Project to a DTO (or anonymous type for quick, local use) for any read-only query, especially API responses — it reduces data transfer, avoids unnecessary change-tracking overhead, and can avoid needing `Include()` for related data you only need a few fields from. Load full tracked entities specifically when the query result is going to be modified and saved.

## Common Mistakes

- Loading full entities and shaping them into a DTO afterward in application code, missing the reduced-data-transfer benefit of projecting directly in the query.
- Assuming a projected DTO is trackable and attempting to modify and save it back through the context.
- Over-relying on `Include()` for related data when a targeted projection pulling only the needed related fields would generate simpler, more efficient SQL.
- Projecting to an entity type instead of a dedicated DTO for an API response, risking exposing internal fields never meant to leave the domain layer.

## Common Interview Questions

### Basic
- What does `Select()` do in an EF Core query, and how does it affect the generated SQL?
- Can a projected result be modified and saved back to the database?

### Intermediate
- How does projecting related data differ from using `Include()`?
- Why does projecting to a DTO reduce data transfer compared to loading a full entity?

### Advanced
- How would you decide, for a given read endpoint, whether to project to a DTO or load a full tracked entity?
- How does EF Core translate a projection that includes a related entity's property into SQL?

### Follow-up Questions
- Is a projected result ever tracked by the change tracker, even without `AsNoTracking()`?
- Can `Select()` be combined with `Where()` and `OrderBy()` in the same composed query?

### Code Prediction
Given `context.Orders.Select(o => new { o.Id, o.Customer.Name }).ToListAsync()` with no explicit `Include()`, does this query generate a join to the `Customers` table? Why or why not?

## Practical Tasks

- Convert a query loading full entities and mapping them to DTOs in memory into a direct projection query, and compare the generated SQL.
- Write a projection pulling a related entity's specific field without using `Include()`.
- Identify an API endpoint returning a full entity directly and refactor it to project to a dedicated response DTO.

## Readiness Criteria

Use projection to reduce fetched data and avoid unnecessary tracking, project related data without a full `Include()` when only specific fields are needed, and choose between projecting to a DTO and loading a full tracked entity appropriately.

## References

### Microsoft Learn

- [Query data with EF Core - projection](https://learn.microsoft.com/ef/core/querying/select)
