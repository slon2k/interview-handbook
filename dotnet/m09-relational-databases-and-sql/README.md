# Module 9 - Relational Databases and SQL

**Status:** Complete  
**Priority:** Critical  
**Prerequisites:** None (framework-agnostic; can be studied independently of Modules 2-8).

## Scope

This module covers relational database design and SQL fundamentals — schema design, keys and relationships, normalization, querying (joins, aggregation, subqueries, window functions), transactions and concurrency, indexing and performance, and injection safety. It's deliberately learned before [Module 10 - Entity Framework Core](../m10-entity-framework-core/README.md), the same way [Module 7 - HTTP, REST, and API Design](../m07-http-rest-api-design/README.md) precedes [Module 8 - ASP.NET Core Fundamentals](../m08-aspnet-core-fundamentals/README.md): strong data-access decisions require understanding what's actually happening underneath the ORM, not just the ORM's API surface.

Examples use standard SQL with SQL Server (T-SQL) and PostgreSQL syntax noted where they diverge, since either is a reasonable pairing with a .NET backend.

## Learning Outcomes

By the end of this module, you should be able to:

- Design a normalized relational schema with appropriate keys, relationships, and constraints, and justify a deliberate denormalization decision when one is warranted.
- Write joins, grouped aggregations, subqueries, CTEs, and window functions fluently, and predict their output including edge cases involving `NULL` and ties.
- Explain ACID guarantees and choose an appropriate transaction isolation level for a given scenario.
- Design and reason about indexes, and read an execution plan to diagnose a slow query rather than guessing.
- Prevent SQL injection structurally through parameterized queries, including within ORM raw-SQL escape hatches.

## Topics

### 1. Schema Design

- [Tables, rows, and schemas](tables-rows-and-schemas.md)
- [Primary and foreign keys, and relationships](keys-and-relationships.md)
- [Normalization and denormalization trade-offs](normalization-and-denormalization.md)
- [Constraints](constraints.md)

### 2. Querying

- [Inner and outer joins](joins.md)
- [Grouping and aggregation](grouping-and-aggregation.md)
- [Subqueries and CTEs](subqueries-and-ctes.md)
- [Window functions](window-functions.md)
- [NULL handling and three-valued logic](null-handling.md)
- [Views and stored procedures](views-and-stored-procedures.md)

### 3. Transactions and Performance

- [Transactions and ACID](transactions-and-acid.md)
- [Isolation levels](isolation-levels.md)
- [Indexes](indexes.md)
- [Query execution plans](query-execution-plans.md)
- [Pagination (SQL-level)](pagination.md)

### 4. Security

- [SQL injection prevention](sql-injection-prevention.md)

## Scope Boundaries

- ORM-specific concerns — `DbContext`, change tracking, migrations, query translation — belong in [Module 10 - Entity Framework Core](../m10-entity-framework-core/README.md); this module covers the underlying SQL and database behavior independent of any particular ORM.
- API-contract-level pagination (how a client expresses "which page" over HTTP) belongs in [Module 7 - HTTP, REST, and API Design](../m07-http-rest-api-design/README.md); this module covers the SQL mechanics (`OFFSET`/`FETCH`, keyset queries) underneath it.
- NoSQL/document-store trade-offs belong in Module 14 - Architecture and System Design Fundamentals (not yet published); this module is deliberately scoped to relational databases only.
- Deep performance tuning and database-level observability belong in Module 13 - Performance, Diagnostics, and Observability (not yet published); this module covers indexing and execution plans at a practical, everyday level.

## Suggested Learning Sequence

1. Tables, keys, relationships, normalization, and constraints.
2. Joins, grouping/aggregation, subqueries/CTEs.
3. Window functions and NULL handling — the two areas with the sharpest interview edge cases.
4. Views and stored procedures.
5. Transactions, ACID, and isolation levels.
6. Indexes, execution plans, and pagination.
7. SQL injection prevention.

## Practical Deliverables

- Design a normalized schema for a small domain (e.g., an order management system) with appropriate keys, relationships, and constraints.
- Write a set of queries covering joins, grouped aggregation, a recursive CTE, and a "top N per group" window function query.
- Reproduce and fix the classic `NOT IN`/`NULL` subquery trap and the `LEFT JOIN` filtered-in-`WHERE` trap.
- Wrap a multi-statement business operation in a transaction with correct rollback behavior.
- Diagnose a slow query using its execution plan and fix it with an appropriate index.
- Identify and fix a SQL-injection vulnerability in a piece of raw ADO.NET code.

## Interview Coverage

Each topic should include:

- Basic questions for terminology and syntax.
- Intermediate questions involving common usage and trade-offs.
- Advanced questions involving performance, concurrency, and edge-case correctness.
- Follow-up questions that test precise understanding rather than memorized definitions.
- Code-prediction questions grounded in concrete SQL examples, since this module produces some of the sharpest "what does this query actually return" interview questions (NULL comparisons, join filtering, tie-handling in window functions).

## References

### Other

- [PostgreSQL documentation](https://www.postgresql.org/docs/current/)
- [SQL Server T-SQL reference](https://learn.microsoft.com/sql/t-sql/language-reference)
