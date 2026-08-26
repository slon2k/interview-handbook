# Module 10 - Entity Framework Core

**Status:** Complete  
**Priority:** Critical  
**Prerequisites:** [Relational Databases and SQL](../m09-relational-databases-and-sql/README.md)

## Scope

This module covers EF Core: how it maps objects to relational data, translates LINQ into SQL, tracks changes, and manages persistence concerns like migrations, concurrency, and transactions. It's learned after Module 9 deliberately — every performance and correctness issue here (N+1 queries, query translation limits, transaction scope) is easier to reason about once the underlying SQL behavior is already familiar.

## Learning Outcomes

By the end of this module, you should be able to:

- Explain `DbContext` as a unit of work, and correctly scope its lifetime (including pooling) to avoid captive-dependency bugs.
- Configure entities and relationships via the Fluent API where convention and data annotations fall short.
- Reason precisely about query translation, projection, and the three loading strategies — and diagnose N+1 queries and cartesian-explosion problems from their symptoms.
- Review and safely deploy migrations, including data migrations.
- Implement transactions, optimistic concurrency, global query filters, and bulk operations correctly.
- Choose an appropriate testing strategy (fake repository, in-memory provider, SQLite, or Testcontainers) for a given test's fidelity needs.

## Topics

### 1. DbContext Fundamentals

- [DbContext and DbSet](dbcontext-and-dbset.md)
- [DbContext lifetime, thread safety, and pooling](dbcontext-lifetime-and-pooling.md)
- [Entity configuration](entity-configuration.md)
- [Relationships](relationships.md)
- [Code-first migrations](migrations.md)

### 2. Querying and Change Tracking

- [Change tracking](change-tracking.md)
- [AsNoTracking](asnotracking.md)
- [Query translation and IQueryable\<T\>](query-translation-and-iqueryable.md)
- [Projection](projection.md)
- [Eager, explicit, and lazy loading](loading-strategies.md)
- [N+1 queries](n-plus-one-queries.md)
- [Split vs. single queries](split-vs-single-queries.md)
- [Global query filters](global-query-filters.md)

### 3. Transactions and Concurrency

- [Transactions in EF Core](transactions.md)
- [Optimistic concurrency](optimistic-concurrency.md)

### 4. Advanced Data Access

- [Raw SQL in EF Core](raw-sql.md)
- [Bulk operations](bulk-operations.md)

### 5. Testing and Architecture

- [Testing EF-based code](testing-ef-based-code.md)
- [Repository pattern in EF Core](repository-pattern-in-ef-core.md)

## Scope Boundaries

- SQL fundamentals — joins, transactions, isolation levels, indexes, execution plans — belong in [Module 9 - Relational Databases and SQL](../m09-relational-databases-and-sql/README.md); this module covers only the EF-Core-specific layer on top of that foundation.
- The general repository pattern trade-off (interface design, testability, when it's worth it at all) belongs in [Module 4 - Object-Oriented Design and Maintainable Code](../m04-oop-design/repository.md); this module covers only the EF-Core-specific interaction (change tracking, `DbContext` lifetime).
- `IQueryable<T>` versus `IEnumerable<T>` as a general C# concept belongs in [Module 3 - Collections, LINQ, and Basic Algorithms](../m03-collections-linq/README.md); this module covers EF Core's specific translation behavior on top of that.
- Captive-dependency and service-lifetime mechanics in general belong in [Module 8 - ASP.NET Core Fundamentals](../m08-aspnet-core-fundamentals/README.md); this module applies that understanding specifically to `DbContext`.

## Suggested Learning Sequence

1. `DbContext`/`DbSet`, lifetime and pooling, entity configuration, relationships, migrations.
2. Change tracking, `AsNoTracking`, query translation, and projection.
3. Loading strategies, N+1 queries, and split vs. single queries — the classic EF Core performance triad.
4. Global query filters, transactions, and optimistic concurrency.
5. Raw SQL and bulk operations.
6. Testing strategies and the EF-Core-specific repository-pattern discussion.

## Practical Deliverables

- Reproduce and fix a captive-dependency bug involving a scoped `DbContext` injected into a singleton.
- Reproduce an N+1 query problem via lazy loading, detect it via query logging, and fix it with eager loading or projection.
- Reproduce and fix the cartesian-explosion problem from including two collection navigations in one query.
- Implement soft delete via a global query filter, including a deliberate `IgnoreQueryFilters()` bypass for an admin report.
- Implement optimistic concurrency with a `RowVersion` token and handle a real conflict.
- Migrate a slow load-mutate-`SaveChanges` bulk operation to `ExecuteUpdate`.
- Set up at least two of the three testing approaches (in-memory provider, SQLite, Testcontainers) for the same repository and compare what each does and doesn't catch.

## Interview Coverage

Each topic should include:

- Basic questions for terminology and API familiarity.
- Intermediate questions involving common usage and configuration trade-offs.
- Advanced questions involving performance diagnosis and production failure modes.
- Follow-up questions that test precise understanding rather than memorized definitions.
- Code-prediction questions grounded in concrete query and SaveChanges examples — this module produces some of the highest-value "why is this slow" and "why didn't this save" interview questions.

## References

### Microsoft Learn

- [Entity Framework Core documentation](https://learn.microsoft.com/ef/core/)
- [EF Core performance](https://learn.microsoft.com/ef/core/performance/)
- [Testing code that uses EF Core](https://learn.microsoft.com/ef/core/testing/)
