# Relationships

## Definition

EF Core maps the relational concepts from Module 9 (foreign keys, one-to-many, many-to-many) onto object-oriented **navigation properties** — references or collections on an entity that let you traverse a relationship in code instead of writing a join by hand.

```csharp
public class Customer
{
    public int Id { get; set; }
    public List<Order> Orders { get; set; } = new(); // one-to-many navigation
}

public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }   // foreign key
    public Customer Customer { get; set; } = null!; // navigation back to the "one" side
}
```

## Alternatives & Trade-offs

Navigation properties let application code express relationships object-graph-style (`customer.Orders`) instead of manually joining and mapping rows, which is a major part of what an ORM buys you. The trade-off is that navigating relationships incorrectly (see `n-plus-one-queries.md` and `loading-strategies.md`) can generate far more SQL than a hand-written join would, if the loading strategy isn't chosen deliberately.

## How It Works

### One-to-many — convention and explicit configuration

```csharp
// By convention, EF Core infers this relationship from CustomerId + the two navigation properties
builder.HasMany(c => c.Orders)
       .WithOne(o => o.Customer)
       .HasForeignKey(o => o.CustomerId);
```

### Many-to-many — EF Core can manage the join table implicitly (5.0+)

```csharp
public class Student
{
    public int Id { get; set; }
    public List<Course> Courses { get; set; } = new();
}
public class Course
{
    public int Id { get; set; }
    public List<Student> Students { get; set; } = new();
}
// EF Core creates and manages a StudentCourse join table automatically, with no explicit entity for it
```

```csharp
// Explicit join entity — needed when the relationship itself carries data (e.g., enrollment date)
public class Enrollment
{
    public int StudentId { get; set; }
    public int CourseId { get; set; }
    public DateTime EnrolledAt { get; set; }
}
builder.HasKey(e => new { e.StudentId, e.CourseId });
```

Use the implicit join table when the many-to-many relationship itself has no extra data; use an explicit join entity the moment it does (like `EnrolledAt` above).

### Optional vs. required relationships

```csharp
public int? CustomerId { get; set; } // nullable FK -> optional relationship
public Customer? Customer { get; set; }

builder.HasOne(o => o.Customer).WithMany(c => c.Orders).IsRequired(false);
```

### Owned entity types — value objects without their own identity/table

```csharp
public class Order
{
    public Address ShippingAddress { get; set; } = null!; // no separate Id, no separate table by default
}
builder.OwnsOne(o => o.ShippingAddress);
```

An owned type maps to columns on the owner's table (or an optional separate table) without being a full entity with its own key and independent lifecycle — a good fit for value objects like `Address` or `Money`.

## Application

Model relationships with navigation properties matching the actual domain shape, configure explicitly via Fluent API whenever convention doesn't infer the intended relationship correctly, and use owned entity types for value objects that don't need independent identity or querying.

## Common Mistakes

- Adding a navigation property in both directions without understanding EF Core's default cascade-delete behavior on the relationship, risking unintended cascading deletes (see the referential-action discussion in Module 9).
- Using an implicit many-to-many join table when the relationship actually needs to carry data, then having to retrofit an explicit join entity later.
- Not configuring required vs. optional relationships explicitly, leaving the database schema's nullability inconsistent with the actual business rule.
- Treating an owned entity type as if it had independent identity, when it's actually tied entirely to its owner's lifecycle.

## Common Interview Questions

### Basic
- What is a navigation property?
- How does EF Core model a many-to-many relationship?

### Intermediate
- When would you need an explicit join entity instead of an implicit many-to-many mapping?
- What's the difference between an owned entity type and a regular related entity?

### Advanced
- How would you configure a relationship where EF Core's default convention doesn't correctly infer the foreign key?
- How do owned entity types affect querying — can you query them independently of their owner?

### Follow-up Questions
- Does deleting a parent entity always cascade to delete its related child entities by default?
- Can a navigation property be one-directional (only on one side of the relationship)?

### Code Prediction
Given `Customer.Orders` and `Order.Customer` as a mutually-navigable one-to-many relationship with no explicit `OnDelete` configuration, what happens by convention when a `Customer` with existing `Orders` is deleted?

## Practical Tasks

- Configure a one-to-many relationship explicitly via Fluent API, including required-ness and delete behavior.
- Model a many-to-many relationship that needs to carry extra data using an explicit join entity.
- Map a value object (like an address) as an owned entity type and query an entity that contains it.

## Readiness Criteria

Model one-to-many, many-to-many, and owned-entity relationships correctly, configure required-ness and delete behavior explicitly, and choose between implicit and explicit join entities appropriately.

## References

### Microsoft Learn

- [Relationships](https://learn.microsoft.com/ef/core/modeling/relationships)
- [Owned entity types](https://learn.microsoft.com/ef/core/modeling/owned-entities)
