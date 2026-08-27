# Test Data Builders

## Definition

A test data builder is a small, fluent helper class that constructs a valid, sensibly-defaulted test object with only the specific fields relevant to a given test overridden — reducing test setup noise and making each test's actual intent (which fields matter) visible at a glance.

```csharp
public class OrderBuilder
{
    private decimal _subtotal = 100m;
    private decimal _discountRate = 0m;
    private OrderStatus _status = OrderStatus.Pending;

    public OrderBuilder WithSubtotal(decimal subtotal) { _subtotal = subtotal; return this; }
    public OrderBuilder WithDiscount(decimal rate) { _discountRate = rate; return this; }
    public OrderBuilder WithStatus(OrderStatus status) { _status = status; return this; }

    public Order Build() => new Order(_subtotal, _discountRate) { Status = _status };
}
```

## Alternatives & Trade-offs

Constructing test objects directly with every field specified inline is explicit but noisy — a test about discount calculation shouldn't need to specify a customer name, an address, and a dozen other irrelevant fields just to compile. A test data builder defaults everything sensibly and lets each test override only what it actually cares about, at the cost of one more class to maintain per entity type — worth it once an entity has enough fields that inline construction becomes noisy or repetitive across many tests.

## How It Works

### Without a builder — noisy, and unclear what actually matters to this test

```csharp
[Fact]
public void CalculateTotal_AppliesDiscount()
{
    var order = new Order
    {
        Id = 1,
        CustomerId = 7,
        CustomerName = "Test Customer",
        ShippingAddress = "123 Test St",
        Subtotal = 100m,
        DiscountRate = 0.1m,   // this is the only field this test actually cares about
        Status = OrderStatus.Pending,
        CreatedAt = DateTime.UtcNow
    };
    Assert.Equal(90m, order.CalculateTotal());
}
```

### With a builder — the test's actual intent is visible at a glance

```csharp
[Fact]
public void CalculateTotal_AppliesDiscount()
{
    var order = new OrderBuilder().WithSubtotal(100m).WithDiscount(0.1m).Build();
    Assert.Equal(90m, order.CalculateTotal());
}
```

Anyone reading this test immediately sees that subtotal and discount rate are what matter — everything else is a sensible, irrelevant default the builder already handles.

### Builders for related object graphs

```csharp
public class OrderBuilder
{
    private readonly List<OrderItemBuilder> _items = new();
    public OrderBuilder WithItem(Action<OrderItemBuilder> configure)
    {
        var itemBuilder = new OrderItemBuilder();
        configure(itemBuilder);
        _items.Add(itemBuilder);
        return this;
    }
    public Order Build() => new Order { Items = _items.Select(b => b.Build()).ToList() };
}

var order = new OrderBuilder()
    .WithItem(i => i.WithProduct("SKU-1").WithQuantity(2))
    .Build();
```

## Application

Introduce a test data builder once an entity's constructor or object-initializer setup becomes noisy or repeated across many tests, especially for entities with several fields where most tests only care about one or two. Default every field to a valid, sensible value so a test only needs to override what's actually relevant to it.

## Common Mistakes

- Specifying every field inline in every test, obscuring which fields actually matter to that specific test's assertion.
- Building a test data builder with defaults that are themselves invalid or nonsensical, forcing every test to override fields that should have had a sensible default.
- Over-engineering a builder with excessive flexibility for fields that are never actually varied across the real test suite.
- Duplicating similar object-construction logic across many test files instead of centralizing it in one shared builder.

## Common Interview Questions

### Basic
- What is a test data builder, and what problem does it solve?
- How does a builder make a test's intent clearer compared to inline object construction?

### Intermediate
- How would you design a builder for an entity with a related collection (like order items)?
- When is a test data builder not worth the added class, compared to plain object initializers?

### Advanced
- How would you structure builders for a deep object graph (an order with items, each item with a product) while keeping each builder focused?
- How do test data builders relate to the general Builder pattern covered in Module 4, and what's different about their intent here?

### Follow-up Questions
- Should a test data builder validate the objects it constructs the same way a production constructor would?
- Can test data builders be combined with parameterized tests?

### Code Prediction
Given the noisy inline-construction example above, if a new required field is added to `Order`, how many test files need to change to keep compiling? How does that number differ if a builder with a sensible default for the new field is used instead?

## Practical Tasks

- Build a test data builder for an entity with several fields, and refactor a set of tests to use it instead of inline construction.
- Extend a builder to support configuring a related collection (like order items) fluently.
- Identify duplicated object-construction logic across several test files and consolidate it into one shared builder.

## Readiness Criteria

Design and use test data builders to reduce test setup noise and clarify test intent, and know when introducing one is and isn't worth the added maintenance.

## References

### Other

- [Martin Fowler: ObjectMother (a related pattern)](https://martinfowler.com/bliki/ObjectMother.html)
