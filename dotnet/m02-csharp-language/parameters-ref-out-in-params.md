# Parameters: `ref`, `out`, `in`, and `params`

## Definition

C# normally passes arguments by value. Parameter modifiers change what is passed or how it is used.

- `ref` passes a variable by reference and requires it to be initialized.
- `out` passes a variable by reference and requires the method to assign it.
- `in` passes a read-only reference, commonly for large value types.
- `params` permits a variable number of arguments.

```csharp
static void Increment(ref int value) => value++;
static bool TryParse(string text, out int value) => int.TryParse(text, out value);
static int Read(in int value) => value;
static int Sum(params int[] values) => values.Sum();
```

## Alternatives & Trade-offs

Ordinary parameters are simplest and safest. Use `ref`, `in`, or `out` only when the API contract benefits from aliasing, multiple outputs, or avoiding a meaningful copy. Prefer a result object or tuple when several outputs form a coherent result.

`params` improves call-site readability but may allocate an array at the call site. A collection or span-based overload may be better on a hot path.

## How It Works

`ref` and `out` allow the callee to access the caller's storage. A `ref` argument must be definitely assigned before the call; an `out` argument need not be initialized, but every normal return path must assign it.

`in` prevents assignment through the parameter. The compiler may create a defensive copy in some cases, especially when calling non-readonly members on a readonly value.

`params` is represented as an array parameter. The compiler creates the array for multiple arguments unless an existing array is passed.

## Application

- Use `ref` for intentional in-place mutation or efficient large-struct APIs.
- Use `out` for Try-pattern APIs where failure is expected.
- Use `in` for large readonly structs when profiling shows copying matters.
- Use `params` for convenient variadic APIs.
- Use tuples or records when named results are clearer than multiple output parameters.

## Common Mistakes

- Passing an expression instead of a variable to `ref`.
- Assuming reference-type parameters themselves are passed by reference.
- Reading an `out` parameter before assigning it.
- Using `ref` to hide broad mutation from callers.
- Adding `in` everywhere without measuring copy costs.
- Allocating repeatedly through `params` in performance-critical loops.

## Common Interview Questions

### Basic

- What is the difference between `ref` and `out`?
- What does `in` mean on a parameter?
- What problem does `params` solve?
- Must a `ref` argument be initialized?

### Intermediate

- Can a method return multiple values without `out` parameters?
- How are reference-type arguments passed by default?
- What happens when a `params` method receives an array?
- Can `ref` parameters be used with properties?

### Advanced

- How do aliases created by `ref` affect API reasoning and thread safety?
- When can an `in` parameter cause a defensive copy?
- How do `ref struct` restrictions affect parameter design?
- How would you expose allocation-free parsing with spans?
- What are the binary compatibility implications of changing a parameter modifier?
- How do `ref`, `in`, and `out` interact with overload resolution?
- When is a result record preferable to multiple `out` values?
- How can `params` affect allocations and overload selection?
- What lifetime rules apply to references returned by `ref` methods?
- How would you benchmark a by-value API against a by-reference API?

### Follow-up Questions

- Can `out` parameters be nullable?
- Can an argument passed with `ref` be reassigned by the method?
- Why is `ref` not allowed on an auto-property expression?
- What is `ref readonly`?
- When does a `params` call allocate?

### Code Prediction

What is printed?

```csharp
int value = 1;
Increment(ref value);
Console.WriteLine(value);
```

What is printed?

```csharp
int value;
TryParse("42", out value);
Console.WriteLine(value);
```

## Practical Tasks

### Try Pattern

Implement `TryReadPort(string text, out int port)` with validation and a clear failure result.

### Struct Performance

Create a large readonly struct and compare by-value, `in`, and `ref` methods using a benchmark.

### API Review

Refactor an API with three `out` parameters into a named result type. Explain the trade-off.

## Readiness Criteria

You should be able to describe the caller/callee storage model, choose the right modifier, explain definite assignment, and identify allocation or aliasing risks.

## References

### Microsoft Learn

- [Method parameters](https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/method-parameters)
- [Reference parameters](https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/method-parameters#reference-parameters)
- [Output parameters](https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/method-parameters#output-parameters)
- [Parameter arrays](https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/method-parameters#parameter-arrays)
