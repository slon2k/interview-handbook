# Method Overloading versus Overriding

## Definition

Overloading defines multiple methods with the same name but different signatures. Overriding replaces inherited virtual behavior.

## Alternatives & Trade-offs

Overloads improve call-site convenience but can create ambiguous or surprising conversions. Overrides support runtime polymorphism but couple behavior to inheritance.

## How It Works

Overload resolution occurs at compile time using the static types and arguments. Override dispatch occurs at runtime using the object's actual type.

## Application

Use overloads for related input forms and overrides for genuine specialization of a base contract.

## Common Mistakes

- Expecting overloads to dispatch by runtime type.
- Forgetting `virtual`, `override`, or `new` semantics.
- Creating overloads with ambiguous optional parameters.

## Common Interview Questions

### Basic
- What is overloading?
- What is overriding?

### Intermediate
- When is each selected?
- What is method hiding?

### Advanced
- How do optional arguments and generic inference affect overload resolution?
- How can overload additions break source compatibility?
- How does virtual dispatch interact with interfaces?

### Follow-up Questions
- Can static methods be overridden?
- What does `sealed override` do?

### Code Prediction
Which method is selected when a derived object is stored in a base-typed variable?

## Practical Tasks

- Predict overload and override behavior for a small hierarchy.
- Refactor ambiguous overloads into named operations.

## Readiness Criteria

Explain static type, runtime dispatch, method signatures, hiding, and API compatibility risks.

## References

### Microsoft Learn

- [Method overloading](https://learn.microsoft.com/dotnet/csharp/programming-guide/classes-and-structs/method-overloading)
- [Override modifier](https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/override)
