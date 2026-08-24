# `IEnumerable<T>` versus `IQueryable<T>`

## Definition

`IEnumerable<T>` represents in-memory or pull-based enumeration. `IQueryable<T>` represents a query expression that a provider may translate and execute elsewhere, such as a database.

```csharp
IEnumerable<User> inMemory = users.Where(user => user.IsActive);
IQueryable<User> queryable = db.Users.Where(user => user.IsActive);
```

## Alternatives & Trade-offs

Use `IEnumerable<T>` when processing objects in the application process. Use `IQueryable<T>` only across a deliberate provider boundary where expression translation is supported and understood.

`IQueryable<T>` can push filtering and projection to a database, but provider translation, round trips, generated SQL, and supported operations must be considered. Do not expose provider-specific query composition casually from application boundaries.

## How It Works

Enumerable LINQ operators compile to delegates and execute against objects. Queryable operators build expression trees; a provider inspects those trees and decides how to execute them.

```csharp
IQueryable<User> query = db.Users
    .Where(user => user.IsActive)
    .Select(user => new User { Id = user.Id });

List<User> result = query.ToList();
```

The query generally executes at a terminal operation such as `ToList`, `First`, or `Count`. Provider behavior belongs with the data-access module, especially EF Core.

## Application

- Filter and project in the database before materialization.
- Keep provider-specific queries near the data-access boundary.
- Inspect generated SQL for important queries.
- Materialize before switching to in-memory-only operations.
- Return application models rather than leaking `IQueryable<T>` unnecessarily.

## Common Mistakes

- Calling `AsEnumerable` too early and moving work into memory.
- Calling `ToList` too early and loading unnecessary rows.
- Assuming every .NET method can be translated by a provider.
- Returning `IQueryable<T>` from a service without controlling composition.
- Enumerating a query repeatedly and issuing multiple database calls.

## Common Interview Questions

### Basic

- What is the difference between `IEnumerable<T>` and `IQueryable<T>`?
- What is an expression tree?
- When does an `IQueryable<T>` query execute?

### Intermediate

- What does `AsEnumerable` do?
- Why can `ToList` placement affect performance?
- What happens when a provider cannot translate an expression?

### Advanced

- How do expression trees enable provider translation?
- How can accidental client-side evaluation affect correctness and performance?
- How should query composition be bounded at an application boundary?
- How do projection and pagination reduce database and network costs?
- How would you test generated query behavior without coupling every test to SQL text?
- What security risks arise when accepting arbitrary query expressions?

### Follow-up Questions

- Does `IQueryable<T>` always mean database execution?
- What is the effect of calling `AsEnumerable`?
- Why should filtering usually happen before materialization?
- Which module should cover EF Core query translation?

### Code Prediction

Where does the filter execute?

```csharp
var result = db.Users
    .Where(user => user.IsActive)
    .Select(user => user.Name)
    .ToList();
```

## Practical Tasks

- Compare the SQL and data volume for filtering before versus after materialization.
- Identify an expression that a provider cannot translate.
- Refactor a service so provider-specific query composition stays in the data-access layer.

## Readiness Criteria

Explain delegates versus expression trees, query execution boundaries, materialization timing, translation limits, and why detailed provider behavior belongs in EF Core.

## References

### Microsoft Learn

- [Query execution](https://learn.microsoft.com/dotnet/csharp/linq/standard-query-operators/query-execution)
- [Expression trees](https://learn.microsoft.com/dotnet/csharp/advanced-topics/expression-trees/)
- [IQueryable<T> interface](https://learn.microsoft.com/dotnet/api/system.linq.iqueryable-1)
- [IEnumerable<T> interface](https://learn.microsoft.com/dotnet/api/system.collections.generic.ienumerable-1)
