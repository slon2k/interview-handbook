# Builder Pattern

## Definition

The Builder pattern separates the construction of a complex object from its representation, assembling it step by step instead of through one large constructor.

```csharp
var request = new HttpRequestBuilder()
    .WithMethod("POST")
    .WithUrl("/orders")
    .WithHeader("Authorization", "Bearer token")
    .WithBody(payload)
    .Build();
```

## Alternatives & Trade-offs

Compared to a large constructor with many optional parameters (a "telescoping constructor"), a builder makes construction readable and lets each step validate incrementally. C# object initializers and named/optional parameters cover many of the same cases with far less code, so a full Builder class is best reserved for objects with genuinely complex construction — multiple required steps, ordering constraints, or validation that spans several fields.

## How It Works

### The problem: telescoping constructors

```csharp
public HttpRequest(string method, string url, string? auth, string? contentType, byte[]? body, int timeoutMs)
```

Callers must remember parameter order and pass `null`/defaults for anything unused — error-prone and hard to read at the call site.

### Builder solution

```csharp
public sealed class HttpRequestBuilder
{
    private string _method = "GET";
    private string _url = "";
    private readonly Dictionary<string, string> _headers = new();
    private byte[]? _body;

    public HttpRequestBuilder WithMethod(string method) { _method = method; return this; }
    public HttpRequestBuilder WithUrl(string url) { _url = url; return this; }
    public HttpRequestBuilder WithHeader(string key, string value) { _headers[key] = value; return this; }
    public HttpRequestBuilder WithBody(byte[] body) { _body = body; return this; }

    public HttpRequest Build()
    {
        if (string.IsNullOrEmpty(_url)) throw new InvalidOperationException("Url is required");
        return new HttpRequest(_method, _url, _headers, _body);
    }
}
```

Each `With...` method returns `this`, enabling the fluent chain, and `Build()` is where cross-field validation happens.

### When object initializers are enough

```csharp
// No builder needed: no complex validation, no required ordering
var order = new Order
{
    CustomerId = 42,
    Items = { new OrderItem("SKU-1", 2) }
};
```

## Application

Reach for a builder when an object has several optional parts, construction-time validation that spans multiple fields, or a natural step-by-step assembly process (HTTP requests, SQL query builders, test data builders for complex aggregates). For simple DTOs, prefer object initializers or a primary constructor.

## Common Mistakes

- Building a full Builder class for a type with two or three simple properties, where an object initializer would be clearer.
- Forgetting to validate in `Build()`, letting an incomplete or inconsistent object escape into the rest of the application.
- Making builder methods mutate and return `this` inconsistently, breaking the fluent chain for some callers.
- Reusing one builder instance across multiple builds without resetting state, causing values to leak between objects.

## Common Interview Questions

### Basic
- What problem does the Builder pattern solve?
- How does a builder differ from a constructor with optional parameters?

### Intermediate
- Why might object initializers make a builder unnecessary for a given type?
- Where should validation live in a builder — in each `With...` method, or in `Build()`?

### Advanced
- How would you make a builder immutable (each `With...` call returns a new builder instance) versus mutable, and what are the trade-offs?
- How do builders fit into test-data-generation patterns (test object builders / object mothers)?

### Follow-up Questions
- Can a builder enforce that certain steps happen in a required order?
- Is a fluent API always implemented via the Builder pattern?

### Code Prediction
```csharp
var builder = new HttpRequestBuilder().WithMethod("POST");
var reqA = builder.WithUrl("/a").Build();
var reqB = builder.WithUrl("/b").Build();
```
If `HttpRequestBuilder` is mutable and reused, what url does `reqA` end up with after `reqB` is built? What design change avoids this?

## Practical Tasks

- Build a fluent `HttpRequestBuilder` with required-field validation in `Build()`.
- Convert a constructor with six optional parameters into a builder-based API.
- Design a test-data builder for an `Order` aggregate used across multiple unit tests.

## Readiness Criteria

Judge when a builder earns its complexity versus when object initializers suffice, implement a fluent builder with proper validation, and avoid state-leak bugs from builder reuse.

## References

### Microsoft Learn

- [Object and collection initializers](https://learn.microsoft.com/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers)
- [Builder pattern in the microservices architecture guide](https://learn.microsoft.com/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/implement-value-objects)
