# Guard Clauses

## Definition

A guard clause is an early, explicit check at the start of a method that validates preconditions and exits (via `return` or `throw`) before the main logic runs, avoiding deeply nested conditionals.

```csharp
public decimal CalculateDiscount(Order order)
{
    if (order is null) throw new ArgumentNullException(nameof(order));
    if (order.Items.Count == 0) return 0m;

    // main logic, unindented, with preconditions already handled
    return order.Total * 0.1m;
}
```

## Alternatives & Trade-offs

Compared to nested `if`/`else` structures, guard clauses flatten a method's control flow, making the "happy path" the least-indented, most obvious code. The trade-off is style, not behavior: guard clauses have multiple exit points from a method, which some style guides discourage in favor of a single return statement. In practice, for precondition checks specifically, most modern C# style guides favor guard clauses for readability.

## How It Works

### Nested conditionals (harder to follow)

```csharp
public decimal CalculateDiscount(Order order)
{
    if (order != null)
    {
        if (order.Items.Count > 0)
        {
            return order.Total * 0.1m;
        }
        else
        {
            return 0m;
        }
    }
    else
    {
        throw new ArgumentNullException(nameof(order));
    }
}
```

### Guard clauses (flattened)

```csharp
public decimal CalculateDiscount(Order order)
{
    if (order is null) throw new ArgumentNullException(nameof(order));
    if (order.Items.Count == 0) return 0m;

    return order.Total * 0.1m;
}
```

Both versions behave identically; the second is easier to scan because each precondition is handled and dismissed in one line before the reader needs to think about the main calculation.

### Guard clauses with modern C#

```csharp
public void Process(Order order)
{
    ArgumentNullException.ThrowIfNull(order);      // .NET 6+ helper for a common guard
    if (order.Total < 0) throw new ArgumentOutOfRangeException(nameof(order));

    // main logic
}
```

## Application

Use guard clauses at the top of public methods and constructors to validate arguments, required state, and preconditions before the main body executes — especially useful in domain entities and value objects to keep invalid states from being constructed at all.

## Common Mistakes

- Validating deep inside a method after other logic has already run, so a thrown exception may leave partial side effects behind.
- Using guard clauses to hide actual business logic branches — guard clauses are for preconditions/invariants, not for the core decision-making of the method.
- Repeating the same guard clause logic across many methods instead of centralizing it (e.g., in a constructor, a validator, or a shared helper).
- Swallowing an invalid precondition silently (returning a default value) when throwing would surface a real bug earlier.

## Common Interview Questions

### Basic
- What is a guard clause, and what problem does it solve?
- How do guard clauses affect method readability compared to nested conditionals?

### Intermediate
- Where should argument validation happen in a constructor versus a factory method?
- What's the difference between a guard clause that throws and one that returns early?

### Advanced
- How do guard clauses support making invalid object states unconstructable (tying into Encapsulation and Immutability topics)?
- How would you centralize repeated guard-clause logic across many methods without hiding validation from readers?

### Follow-up Questions
- Should a guard clause ever be used to suppress an error instead of surfacing it?
- Do guard clauses conflict with the "single return per method" style guideline?

### Code Prediction
Given the nested-conditional version of `CalculateDiscount` above, what would happen if `order` is `null`—does it throw immediately, or does the null-check happen after other work? Compare that to the guard-clause version.

## Practical Tasks

- Refactor a deeply nested method with 3+ levels of `if`/`else` into one using guard clauses.
- Add guard clauses to a constructor so an entity cannot be constructed in an invalid state.
- Identify a case where validation happens too late in a method (after side effects), and move it earlier.

## Readiness Criteria

Refactor nested conditionals into guard clauses without changing behavior, apply guard clauses to enforce invariants at construction time, and judge when early validation prevents partial side effects.

## References

### Microsoft Learn

- [ArgumentNullException.ThrowIfNull method](https://learn.microsoft.com/dotnet/api/system.argumentnullexception.throwifnull)
- [Code quality rules for argument validation](https://learn.microsoft.com/dotnet/fundamentals/code-analysis/quality-rules/ca1062)
