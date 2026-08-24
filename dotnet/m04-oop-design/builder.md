# Builder Pattern

## Definition

Builder separates construction of a complex object from its representation and can make multi-step creation readable.

## Alternatives & Trade-offs

Object initializers, records, and constructors are simpler for small objects. Builders help when construction has many optional values, validation, or ordered steps.

## How It Works

A builder collects configuration and produces the final object, often validating at `Build` time. A builder should not allow an invalid intermediate state to escape as a finished object.

## Application

Use for complex requests, test data, configuration, and objects with many optional settings.

## Common Mistakes

- Creating builders for simple types.
- Reusing a mutable builder across threads.
- Delaying all validation until a confusing build failure.

## Common Interview Questions

### Basic
- What problem does Builder solve?
- When is a builder useful?

### Intermediate
- How does Builder differ from a Factory?
- Where should validation occur?

### Advanced
- How can staged builders enforce construction order at compile time?
- What are builder reuse and thread-safety risks?
- When do records and init properties remove the need for Builder?

### Follow-up Questions
- Should `Build` return a new object every time?
- Can a builder be immutable?

### Code Prediction
When should required configuration validation run in a builder?

## Practical Tasks

- Design a builder for a complex immutable request.
- Compare it with a record and object initializer for readability and safety.

## Readiness Criteria

Choose Builder only for meaningful construction complexity, validate at the right boundary, and explain factory and initializer alternatives.

## References

### Microsoft Learn

- [Object initializers](https://learn.microsoft.com/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers)
- [Records](https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/record)
