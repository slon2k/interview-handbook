# Decorator Pattern

## Definition

Decorator adds behavior to an object by wrapping it behind the same interface.

## Alternatives & Trade-offs

Decorators compose behavior without subclass explosion. Middleware or a direct method may be clearer for linear pipelines or one behavior.

## How It Works

A decorator receives the component, performs work before or after delegation, and returns the same contract.

## Application

Logging, caching, authorization, retries, metrics, and validation are common decorator behaviors.

## Common Mistakes

- Changing the wrapped contract unexpectedly.
- Making decorator ordering undocumented.
- Creating decorators with hidden global state.

## Common Interview Questions

### Basic
- What does Decorator solve?
- How does it differ from inheritance?

### Intermediate
- Why does ordering matter?
- How does Decorator support the Open/Closed Principle?

### Advanced
- How should decorators handle exceptions and cancellation?
- How can nested decorators affect observability and performance?
- When is middleware a better fit?

### Follow-up Questions
- Can decorators be stacked?
- How do you test ordering?

### Code Prediction
Which behavior runs first when decorators wrap one another?

## Practical Tasks

- Add logging and caching decorators to a service.
- Document and test decorator ordering and failure behavior.

## Readiness Criteria

Implement composable wrappers, explain ordering and contract preservation, and distinguish decorators from middleware and inheritance.

## References

### Microsoft Learn

- [Decorator design pattern](https://learn.microsoft.com/dotnet/standard/design-patterns/decorator)
