# Delegates, Lambdas and Closures

## Definition

A delegate is a type-safe object that represents a method signature. It can reference static or instance methods and can be passed as data.

```csharp
Func<int, int> square = value => value * value;
Action<string> write = Console.WriteLine;
Predicate<int> isPositive = value => value > 0;
```

A lambda expression is a concise way to create a delegate or expression tree. A closure is a lambda that captures variables from its surrounding scope.

## Alternatives & Trade-offs

Use delegates for callbacks and behavior parameters. Use interfaces when behavior has several operations, state, or a stable object-oriented contract. Use expression trees when a provider must inspect the operation rather than execute it directly.

Lambdas improve local readability, but captured state can extend lifetimes and create allocations. Prefer named methods when the logic is substantial or reused.

## How It Works

A delegate contains a target and method pointer. A lambda that captures state is compiled into a generated object containing the captured variables.

```csharp
Func<int> counter = CreateCounter();

static Func<int> CreateCounter()
{
    int count = 0;
    return () => ++count;
}
```

The returned delegate keeps `count` alive after `CreateCounter` returns. A non-capturing lambda may be cached by the compiler or runtime.

## Application

- Pass callbacks to algorithms and framework APIs.
- Use LINQ predicates and selectors.
- Compose validation or transformation functions.
- Handle asynchronous completion with `Func<Task>`.
- Use expression trees for query providers.

## Common Mistakes

- Capturing a loop variable unexpectedly.
- Capturing mutable state in concurrent code.
- Assuming delegate equality always means identical source expressions.
- Creating delegates repeatedly in a hot loop without measuring.
- Using `async void` except for event handlers.
- Hiding complex business logic in nested lambdas.

## Common Interview Questions

### Basic

- What is a delegate?
- What is a lambda expression?
- What are `Func`, `Action`, and `Predicate`?
- What is a closure?

### Intermediate

- How does a captured variable behave after the method returns?
- What is the difference between a delegate and an interface?
- What is a multicast delegate?
- What is the difference between a delegate and an expression tree?

### Advanced

- How are closures represented by the compiler?
- When can delegate creation allocate?
- How does delegate invocation differ from direct method calls?
- What lifetime and memory risks come from captured objects?
- How do static lambdas prevent accidental capture?
- How does variance apply to delegate parameters and return values?
- How do open instance delegates differ from closed instance delegates?
- What are the performance trade-offs of delegates versus function pointers?
- How do async lambdas affect exception propagation?
- How would you test a callback-heavy API deterministically?

### Follow-up Questions

- Can a lambda capture a value type?
- What happens when a delegate has multiple targets?
- Why can `async void` be difficult to test?
- What is a method group conversion?
- How can you avoid capturing a loop variable?

### Code Prediction

What is printed?

```csharp
int factor = 2;
Func<int, int> multiply = value => value * factor;
factor = 3;
Console.WriteLine(multiply(4));
```

## Practical Tasks

### Callback API

Implement a method that accepts a predicate and returns matching items without exposing collection internals.

### Closure Review

Find and fix an accidental loop-variable capture in a set of callbacks.

### Composition

Compose validation delegates and define how failures are reported.

## Readiness Criteria

You should be able to define and invoke delegates, explain lambda conversion and closure lifetime, choose delegates versus interfaces, and identify allocation and concurrency risks.

## References

### Microsoft Learn

- [Delegates](https://learn.microsoft.com/dotnet/csharp/programming-guide/delegates/)
- [Lambda expressions](https://learn.microsoft.com/dotnet/csharp/language-reference/operators/lambda-expressions)
- [Closures](https://learn.microsoft.com/dotnet/csharp/language-reference/operators/lambda-expressions#capture-of-outer-variables-and-variable-scope-in-lambda-expressions)
- [Expression trees](https://learn.microsoft.com/dotnet/csharp/advanced-topics/expression-trees/)
