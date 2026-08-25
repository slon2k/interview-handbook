# Normalization and Denormalization Trade-offs

## Definition

**Normalization** organizes a schema to minimize data duplication and prevent update anomalies, typically described through **normal forms** (1NF, 2NF, 3NF being the common practical targets). **Denormalization** deliberately reintroduces duplication — often by combining tables or storing computed/redundant values — to optimize read performance at the cost of write complexity and potential inconsistency.

```sql
-- Normalized: customer name stored once, referenced by ID
CREATE TABLE Customers (Id INT PRIMARY KEY, Name NVARCHAR(200));
CREATE TABLE Orders (Id INT PRIMARY KEY, CustomerId INT, FOREIGN KEY (CustomerId) REFERENCES Customers(Id));

-- Denormalized: customer name duplicated into every order row for faster reads
CREATE TABLE Orders (Id INT PRIMARY KEY, CustomerId INT, CustomerName NVARCHAR(200));
```

## Alternatives & Trade-offs

Normalization eliminates update anomalies — a customer's name lives in exactly one place, so renaming them requires exactly one update and can never leave stale, inconsistent copies elsewhere. Denormalization avoids the join needed to reassemble that data, which matters for read-heavy, latency-sensitive queries, but means every write that changes the duplicated value must update every copy, or accept that the copies can drift out of sync.

## How It Works

### The core anomalies normalization prevents

```sql
-- Unnormalized: customer name repeated in every order row
Orders: (Id, CustomerName, CustomerEmail, Product, Amount)

-- Update anomaly: renaming a customer requires updating every one of their order rows;
-- missing even one leaves inconsistent data with no way to tell which is correct
-- Insertion anomaly: can't record a new customer until they place an order (no separate customer row exists)
-- Deletion anomaly: deleting a customer's only order also deletes all record that the customer ever existed
```

### Third normal form (3NF) — the common practical target

```sql
CREATE TABLE Customers (Id INT PRIMARY KEY, Name NVARCHAR(200), Email NVARCHAR(200));
CREATE TABLE Orders (Id INT PRIMARY KEY, CustomerId INT, Product NVARCHAR(200), Amount DECIMAL(10,2));
```

Each fact is stored in exactly one place; a customer's name lives only in `Customers`, referenced by `CustomerId` elsewhere. Most production schemas target 3NF and stop there — further normal forms exist but address increasingly rare edge cases with diminishing practical benefit.

### Deliberate denormalization for read performance

```sql
-- A reporting table intentionally duplicating data to avoid expensive joins on every read
CREATE TABLE OrderSummary (
    OrderId INT PRIMARY KEY,
    CustomerName NVARCHAR(200),  -- duplicated from Customers, refreshed by a batch job or trigger
    TotalAmount DECIMAL(10,2)
);
```

This trades write complexity (something must keep `CustomerName` in sync) for read simplicity and speed (no join required for a very common, performance-sensitive query).

## Application

Normalize to at least 3NF by default — it's the right starting point for transactional (OLTP) schemas where write consistency matters most. Denormalize deliberately and selectively, usually driven by a measured read-performance problem (a specific slow, high-frequency query) rather than upfront — and have a clear, explicit strategy (trigger, batch job, cache invalidation) for keeping denormalized copies in sync.

## Common Mistakes

- Denormalizing prematurely, before a real, measured performance problem justifies the added write complexity and inconsistency risk.
- Denormalizing without a clear strategy for keeping duplicated data in sync, leading to silent data drift over time.
- Over-normalizing to the point that simple, common queries require many joins across many small tables, hurting both readability and performance without a corresponding consistency benefit.
- Treating normalization as an all-or-nothing choice rather than a spectrum — most real schemas mix mostly-normalized core tables with a few deliberately denormalized reporting or caching tables.

## Common Interview Questions

### Basic
- What is normalization, and what problem does it solve?
- What is denormalization, and why would you deliberately introduce duplication?

### Intermediate
- What are the three classic anomalies normalization prevents (update, insertion, deletion)?
- What normal form do most production schemas target, and why not go further?

### Advanced
- How would you decide whether a specific slow query justifies denormalizing part of a schema?
- How would you keep a denormalized copy of data in sync with its source of truth (triggers, batch jobs, event-driven updates), and what are the trade-offs of each approach?

### Follow-up Questions
- Is denormalization ever appropriate in a system with strict consistency requirements?
- Can a schema be normalized in some parts and denormalized in others?

### Code Prediction
Given the unnormalized `Orders: (Id, CustomerName, CustomerEmail, Product, Amount)` example, what happens if a customer's email is updated in one order row but the update script misses a second row for the same customer? What does querying by the old versus new email now return?

## Practical Tasks

- Normalize a given unnormalized table design to 3NF, identifying the update/insertion/deletion anomalies it fixes.
- Design a deliberately denormalized reporting table for a specific slow query, along with a strategy for keeping it in sync.
- Given a schema, identify one table that would benefit from denormalization and justify the trade-off.

## Readiness Criteria

Normalize a schema to 3NF and explain the anomalies it prevents, and make a deliberate, justified denormalization decision including a sync strategy rather than treating it as a default choice.

## References

### Other

- [PostgreSQL: Database normalization (general reference)](https://www.postgresql.org/docs/current/ddl-constraints.html)
