# Entity Configuration

## Definition

Entity configuration tells EF Core how C# classes map to database tables and columns — names, types, keys, required-ness, indexes — beyond what convention infers automatically. EF Core supports **data annotations** (attributes directly on the entity class) and the **Fluent API** (configuration code in `OnModelCreating` or a separate `IEntityTypeConfiguration<T>` class), which can be combined but the Fluent API takes precedence when both configure the same thing.

```csharp
public class Order
{
    public int Id { get; set; }
    [Required, MaxLength(200)]
    public string CustomerName { get; set; } = "";
}
```

```csharp
public class OrderConfiguration : IEntityTypeConfiguration<Order>
{
    public void Configure(EntityTypeBuilder<Order> builder)
    {
        builder.Property(o => o.CustomerName).IsRequired().HasMaxLength(200);
        builder.HasIndex(o => o.CustomerId);
    }
}
```

## Alternatives & Trade-offs

Data annotations are quick and visible directly on the entity, but mix persistence concerns into a class that might otherwise be a plain domain model, and can't express everything (composite keys, some index configurations, relationship details). The Fluent API via `IEntityTypeConfiguration<T>` keeps persistence mapping entirely separate from the entity class — better for domain models that shouldn't know anything about EF Core — and can express the full range of mapping options, at the cost of configuration living in a different file than the entity itself.

## How It Works

### Convention-based mapping — what EF Core infers automatically

```csharp
public class Order
{
    public int Id { get; set; }          // inferred as primary key, by name + type convention
    public int CustomerId { get; set; }  // inferred as a foreign key if a matching navigation exists
    public decimal Total { get; set; }   // inferred as a required (non-nullable) decimal column
}
```

EF Core's conventions handle a large fraction of typical mapping needs automatically — explicit configuration is for overriding or extending those conventions where the default doesn't fit.

### Organizing configuration with `IEntityTypeConfiguration<T>`

```csharp
public class OrderConfiguration : IEntityTypeConfiguration<Order>
{
    public void Configure(EntityTypeBuilder<Order> builder)
    {
        builder.ToTable("Orders", schema: "sales");
        builder.HasKey(o => o.Id);
        builder.Property(o => o.Total).HasColumnType("decimal(10,2)");
        builder.HasIndex(o => o.CustomerId);
    }
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.ApplyConfiguration(new OrderConfiguration());
    // or, to apply every IEntityTypeConfiguration<T> in an assembly automatically:
    modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
}
```

Keeping each entity's configuration in its own class (rather than one giant `OnModelCreating` method) scales far better as the model grows.

### Value conversions — mapping a type EF Core doesn't natively understand

```csharp
builder.Property(o => o.Status)
    .HasConversion(status => status.ToString(), value => Enum.Parse<OrderStatus>(value)); // enum stored as string
```

### Composite keys — only expressible via the Fluent API

```csharp
builder.HasKey(sc => new { sc.StudentId, sc.CourseId }); // data annotations can't express composite keys
```

## Application

Rely on convention for straightforward, typical mappings. Reach for `IEntityTypeConfiguration<T>` (Fluent API) once a mapping needs anything conventions and data annotations can't express (composite keys, precise column types, indexes, value conversions), or when keeping the entity class free of persistence-specific attributes matters for the domain model's cleanliness.

## Common Mistakes

- Mixing data annotations and Fluent API configuration for the same property inconsistently across a codebase, making it unclear which mapping actually wins (Fluent API does, when both configure the same thing).
- Relying entirely on convention for a column that actually needs a specific type (e.g., letting `decimal` default to a precision that silently truncates values).
- Putting all entity configuration into one large `OnModelCreating` method instead of separate `IEntityTypeConfiguration<T>` classes, making the model configuration hard to navigate as it grows.
- Polluting a domain entity class with EF Core-specific data annotations when the design goal is a persistence-ignorant domain model.

## Common Interview Questions

### Basic
- What's the difference between data annotations and the Fluent API for entity configuration?
- What does `IEntityTypeConfiguration<T>` provide?

### Intermediate
- Why can't a composite key be expressed using data annotations alone?
- What happens if both a data annotation and Fluent API configuration target the same property with different rules?

### Advanced
- How would you organize entity configuration for a large model with 50+ entities to keep it maintainable?
- How would you map an enum property to a readable string column instead of its default integer representation?

### Follow-up Questions
- Does `ApplyConfigurationsFromAssembly` require each configuration class to be registered individually?
- Can entity configuration reference navigation properties to configure relationships (covered further in `relationships.md`)?

### Code Prediction
Given `[MaxLength(200)]` on a property via data annotation, and `builder.Property(x => x.Name).HasMaxLength(100)` via Fluent API for the same property, what maximum length does the generated column actually get?

## Practical Tasks

- Configure a composite primary key using `IEntityTypeConfiguration<T>` for a junction entity.
- Add a value conversion mapping an enum to a string column instead of its default integer value.
- Reorganize entity configuration scattered across data annotations and one large `OnModelCreating` method into separate `IEntityTypeConfiguration<T>` classes.

## Readiness Criteria

Choose between data annotations and Fluent API appropriately, express configurations that annotations can't (composite keys, value conversions), and organize configuration for a growing model.

## References

### Microsoft Learn

- [Entity properties](https://learn.microsoft.com/ef/core/modeling/entity-properties)
- [Creating and configuring a model](https://learn.microsoft.com/ef/core/modeling/)
- [Value conversions](https://learn.microsoft.com/ef/core/modeling/value-conversions)
