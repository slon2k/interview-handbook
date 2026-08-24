# Repository Pattern

## Definition

A Repository provides a domain-oriented interface for accessing or persisting aggregates, hiding storage details from application code. In Domain-Driven Design, an aggregate is a consistency boundary containing a root entity and the objects it controls.

## Alternatives & Trade-offs

A repository can isolate a complex or remote data source. Over EF Core, an extra generic repository may duplicate `DbSet` and LINQ behavior, reduce query flexibility, and obscure generated SQL.

## How It Works

A repository defines operations meaningful to the domain or application. Infrastructure implements those operations and maps persistence models where needed.

## Application

Use a repository when it represents a real boundary, coordinates multiple stores, or protects domain code from infrastructure details. Keep EF Core-specific query behavior in the data-access layer.

## Common Mistakes

- Creating a generic CRUD wrapper over every `DbSet`.
- Returning `IQueryable<T>` without controlling composition.
- Mixing transaction ownership across repository and service layers.
- Hiding important query cost and loading behavior.

## Common Interview Questions

### Basic
- What is the Repository pattern?
- What problem can it solve?

### Intermediate
- Is a repository always needed with EF Core?
- Where should transaction boundaries live?

### Advanced
- How can repositories preserve aggregate invariants?
- How do repositories affect query composition and performance?
- How would you test a repository without making every test a mock?

### Follow-up Questions
- What is the difference between Repository and Unit of Work?
- Should repositories return entities or DTOs?

### Code Prediction
Which abstraction is clearer: `GetById` for a domain need or a generic method exposing every persistence operation?

## Practical Tasks

- Design a repository around an aggregate rather than generic CRUD.
- Compare direct EF Core access with a repository and document the trade-offs.

## Readiness Criteria

Explain repository boundaries, aggregate ownership, query and transaction trade-offs, and when the pattern adds redundant abstraction.

## References

### Microsoft Learn

- [EF Core DbContext](https://learn.microsoft.com/ef/core/dbcontext-configuration/)
- [EF Core change tracking](https://learn.microsoft.com/ef/core/change-tracking/)
