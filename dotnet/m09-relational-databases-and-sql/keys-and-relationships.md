# Primary and Foreign Keys, and Relationships

## Definition

A **primary key** uniquely identifies each row in a table. A **foreign key** is a column (or set of columns) in one table that references a primary key in another, establishing a **relationship** between the two tables and letting the database enforce **referential integrity** — a foreign key value must correspond to an existing row in the referenced table, or be `NULL` if the relationship is optional.

```sql
CREATE TABLE Customers (Id INT PRIMARY KEY IDENTITY, Name NVARCHAR(200) NOT NULL);

CREATE TABLE Orders (
    Id INT PRIMARY KEY IDENTITY,
    CustomerId INT NOT NULL,
    FOREIGN KEY (CustomerId) REFERENCES Customers(Id)
);
```

## Alternatives & Trade-offs

Enforcing referential integrity at the database level (foreign key constraints) guarantees consistency regardless of which application or code path writes to the database — no orphaned `Orders` row can ever point to a deleted `Customer`. The alternative — enforcing it only in application code — is more flexible (easier to bulk-load data, easier to relax temporarily) but leaves the database vulnerable to inconsistency from any code path (a script, a different service, a manual query) that skips the application's validation logic.

## How It Works

### One-to-many

```sql
CREATE TABLE Orders (
    Id INT PRIMARY KEY IDENTITY,
    CustomerId INT NOT NULL,
    FOREIGN KEY (CustomerId) REFERENCES Customers(Id)
);
-- one Customer can have many Orders; each Order belongs to exactly one Customer
```

### Many-to-many via a junction table

```sql
CREATE TABLE Students (Id INT PRIMARY KEY IDENTITY, Name NVARCHAR(200));
CREATE TABLE Courses (Id INT PRIMARY KEY IDENTITY, Title NVARCHAR(200));

CREATE TABLE StudentCourses (
    StudentId INT NOT NULL,
    CourseId INT NOT NULL,
    PRIMARY KEY (StudentId, CourseId),           -- composite primary key
    FOREIGN KEY (StudentId) REFERENCES Students(Id),
    FOREIGN KEY (CourseId) REFERENCES Courses(Id)
);
```

Relational databases have no native way to express many-to-many directly between two tables — a junction (bridge) table is always required, holding one row per pairing.

### Referential actions on delete/update

```sql
FOREIGN KEY (CustomerId) REFERENCES Customers(Id) ON DELETE CASCADE   -- deleting a Customer deletes their Orders
FOREIGN KEY (CustomerId) REFERENCES Customers(Id) ON DELETE RESTRICT  -- deleting a Customer with Orders fails
FOREIGN KEY (CustomerId) REFERENCES Customers(Id) ON DELETE SET NULL  -- deleting a Customer nulls the Order's link (requires the column to allow NULL)
```

Choosing the wrong referential action is a common source of either accidental data loss (`CASCADE` where it wasn't intended) or unexpected constraint-violation errors (`RESTRICT` where cascading was actually expected).

### Natural keys vs. surrogate keys

```sql
-- Surrogate key: a database-generated, meaningless identifier
CREATE TABLE Products (Id INT PRIMARY KEY IDENTITY, Sku NVARCHAR(50) NOT NULL UNIQUE);

-- Natural key: a real-world attribute used directly as the primary key
CREATE TABLE Products (Sku NVARCHAR(50) PRIMARY KEY);
```

Surrogate keys are more common in practice — they're stable even if a "natural" identifier (SKU, email) needs to change later, and they avoid embedding business meaning into something used purely for joining rows.

## Application

Use a surrogate primary key (auto-incrementing integer or GUID) by default, with a separate `UNIQUE` constraint on any natural identifier that also needs uniqueness (like an SKU or email). Enforce relationships with foreign key constraints rather than relying solely on application code, and choose referential actions (`CASCADE`/`RESTRICT`/`SET NULL`) deliberately based on the actual business rule for what should happen on deletion.

## Common Mistakes

- Relying only on application-level checks for referential integrity, allowing orphaned rows to be created by any code path that bypasses that logic.
- Choosing `ON DELETE CASCADE` without considering that it can silently delete large amounts of related data.
- Using a natural key (like an email address) as a primary key, then needing to change it later and discovering every foreign key referencing it must also change.
- Forgetting to index foreign key columns — most databases don't automatically index a foreign key column the way they automatically index a primary key, which can make joins and cascading operations slow (see `indexes.md`).

## Common Interview Questions

### Basic
- What is a primary key, and what is a foreign key?
- How do you model a many-to-many relationship in a relational database?

### Intermediate
- What's the difference between `ON DELETE CASCADE`, `RESTRICT`, and `SET NULL`?
- Why are surrogate keys generally preferred over natural keys?

### Advanced
- What are the risks of relying solely on application-level referential integrity instead of database-enforced foreign keys?
- Why should foreign key columns typically be indexed, even though they aren't automatically?

### Follow-up Questions
- Can a foreign key reference a column that isn't a primary key?
- Is a composite primary key ever appropriate outside of junction tables?

### Code Prediction
Given `FOREIGN KEY (CustomerId) REFERENCES Customers(Id) ON DELETE RESTRICT`, what happens when an attempt is made to delete a `Customer` who still has `Orders` referencing them?

## Practical Tasks

- Design a many-to-many relationship (e.g., students and courses) using a junction table with a composite primary key.
- Choose and justify appropriate `ON DELETE` behavior for three different relationships in a hypothetical schema.
- Identify a schema relying only on application-level referential integrity and add the missing foreign key constraints.

## Readiness Criteria

Design one-to-many and many-to-many relationships correctly, choose appropriate referential actions, and explain why database-enforced foreign keys matter beyond application-level checks.

## References

### Other

- [PostgreSQL: Foreign keys](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-FK)
- [SQL Server: FOREIGN KEY constraints](https://learn.microsoft.com/sql/relational-databases/tables/primary-and-foreign-key-constraints)
