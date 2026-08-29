# Basic Domain-Driven Design Vocabulary

## Definition

Domain-Driven Design (DDD) is a large body of practice for modeling complex business domains; at awareness level, the useful part is a small, precise vocabulary that makes design discussions sharper: **entity** (has identity, persists through change — an `Order`), **value object** (defined entirely by its attributes, no identity — `Money`, an `Address`), **aggregate** (a cluster of entities/value objects treated as one consistency boundary, with a designated **aggregate root** as its only entry point), **bounded context** (a boundary within which a specific model and its terminology apply consistently), and **ubiquitous language** (shared vocabulary between developers and domain experts, reflected directly in the code).

```csharp
public class Order // Entity — has an identity (Id), persists and changes over time
{
    public int Id { get; }
    public Address ShippingAddress { get; set; } // Value Object — no identity, just attributes
    private readonly List<OrderItem> _items = new();
    public IReadOnlyList<OrderItem> Items => _items; // Order is the AGGREGATE ROOT for this cluster
}
```

## Alternatives & Trade-offs

Full DDD (a dedicated modeling process, deep collaboration with domain experts, extensive documentation of bounded contexts) is a significant investment appropriate for genuinely complex business domains where getting the model right matters enormously. At awareness level, just using the vocabulary precisely — distinguishing entities from value objects, respecting aggregate boundaries, being clear about which bounded context a term belongs to — improves design discussions and code clarity without requiring the full ceremony, which is exactly the right scope for a mid-level role.

## How It Works

### Entity vs. value object — already familiar from Module 4, now named precisely

```csharp
// Entity: identity matters — two Orders with identical items are still different orders
public class Order { public int Id { get; } }

// Value object: no identity — two Money instances with the same amount/currency ARE equal (Module 2's equality content)
public record Money(decimal Amount, string Currency);
```

This is the same distinction Module 4's `immutability-in-object-design.md` and Module 2's equality content already covered — DDD just gives it precise, widely-shared names.

### Aggregate and aggregate root — the consistency boundary for changes

```csharp
public class Order // the aggregate root — the ONLY entry point for modifying anything inside this aggregate
{
    private readonly List<OrderItem> _items = new();

    public void AddItem(string sku, int quantity) // external code goes through the root, never touches _items directly
    {
        if (quantity <= 0) throw new ArgumentException(); // invariant enforced at the aggregate boundary
        _items.Add(new OrderItem(sku, quantity));
    }
}
```

`OrderItem` is never modified directly by outside code — everything goes through `Order`, which is exactly what lets `Order` guarantee its invariants hold for the whole cluster, not just itself.

### Bounded context — the same word can mean different things in different contexts, deliberately

```
In the Sales bounded context, "Customer" means someone who has placed an order — name, email,
order history.
In the Support bounded context, "Customer" means someone with support tickets — name, email,
ticket history, SLA tier.

These are DELIBERATELY different models sharing a name, each valid and complete within its own
bounded context — trying to unify them into one "Customer" model serving both would create an
awkward, bloated compromise serving neither context well.
```

This connects directly to the modular-monolith topic's module boundaries — a bounded context is often exactly the boundary a module (or eventually a microservice) should be drawn around.

### Ubiquitous language — code that speaks the business's actual vocabulary

```csharp
// Code using the business's actual terms, not generic technical vocabulary
public void ReconcileShipment(Shipment shipment) { }

// vs. generic, business-vocabulary-free naming that loses this clarity
public void ProcessData(DataObject data) { }
```

## Application

Use entity/value-object/aggregate vocabulary precisely when designing domain models, respecting aggregate boundaries as the unit of consistency for changes. Use "bounded context" to explain why the same business term can validly mean different things in different parts of a system, rather than forcing one unified model everywhere. Reflect the business's actual vocabulary in code and discussion (ubiquitous language) rather than generic technical naming.

## Common Mistakes

- Allowing external code to modify an aggregate's internal entities directly, bypassing the aggregate root and its invariant enforcement.
- Trying to unify a term (like "Customer") into one model across bounded contexts that legitimately need different things from it, producing an awkward compromise.
- Treating full DDD's extensive process and ceremony as required just to use its precise vocabulary usefully in design discussions.
- Using generic, business-vocabulary-free naming in code, losing the clarity that comes from code speaking the same language as the domain experts describing the requirements.

## Common Interview Questions

### Basic
- What's the difference between an entity and a value object?
- What is an aggregate root, and why does it matter?

### Intermediate
- What is a bounded context, and why might the same term validly mean different things in two different bounded contexts?
- What does "ubiquitous language" mean in practice?

### Advanced
- How does the aggregate boundary relate to transactional consistency — what should and shouldn't be included in one aggregate?
- How does the bounded-context concept relate to the modular-monolith and microservices boundaries covered elsewhere in this module?

### Follow-up Questions
- Can a value object ever be mutable?
- Should every entity have exactly one aggregate root, or can aggregates be nested?

### Code Prediction
Given `Order` as an aggregate root containing `OrderItem` entities, if external code holds a reference to a specific `OrderItem` and modifies its quantity directly (bypassing `Order.AddItem`/an equivalent method), what invariant enforcement does this bypass, and why does that matter for the aggregate's consistency guarantee?

## Practical Tasks

- Model a small domain (e.g., a library lending system) identifying its entities, value objects, and at least one aggregate with a clear root.
- Identify a term that would validly have different meanings across two bounded contexts in a hypothetical system, and design each context's model for that term independently.
- Refactor a class that allows direct external modification of its internal collection into a proper aggregate root enforcing its own invariants.

## Readiness Criteria

Use entity/value-object/aggregate/bounded-context vocabulary precisely, design aggregates with the root correctly enforcing invariants, and explain bounded contexts without requiring full DDD process ceremony to apply the idea usefully.

## References

### Other

- [Martin Fowler: DDD_Aggregate](https://martinfowler.com/bliki/DDD_Aggregate.html)
- [Martin Fowler: BoundedContext](https://martinfowler.com/bliki/BoundedContext.html)
