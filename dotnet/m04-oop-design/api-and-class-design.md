# API and Class Design

## Definition

API and class design is about shaping a type's public surface — its constructors, methods, and properties — so it's easy to use correctly and hard to use incorrectly, independent of its internal implementation.

```csharp
// Hard to use correctly: caller must remember the right order and units
public void Schedule(int year, int month, int day, int durationMinutes) { }

// Easier to use correctly: types encode the rules
public void Schedule(DateOnly date, TimeSpan duration) { }
```

## Alternatives & Trade-offs

A richer, more specific public API (dedicated types, guard clauses, restricted constructors) takes more design effort upfront and can feel like over-engineering for a throwaway script, but pays off quickly once a type is used from more than one place — it turns whole classes of bugs into compile errors instead of runtime failures.

## How It Works

### Make invalid states unrepresentable

```csharp
// Both fields are independently settable — nothing stops an inconsistent state
public class DateRange
{
    public DateTime Start { get; set; }
    public DateTime End { get; set; }
}

// Validated at construction — an invalid DateRange cannot exist
public sealed class DateRange
{
    public DateTime Start { get; }
    public DateTime End { get; }

    public DateRange(DateTime start, DateTime end)
    {
        if (end < start) throw new ArgumentException("End must not precede start");
        Start = start;
        End = end;
    }
}
```

### Minimal public surface

```csharp
public class OrderProcessor
{
    public void Process(Order order) { ValidateStock(order); Charge(order); }

    // Implementation details stay private — they're not part of the contract callers depend on
    private void ValidateStock(Order order) { }
    private void Charge(Order order) { }
}
```

Keeping `ValidateStock`/`Charge` private means they can be changed freely without breaking any caller.

### Types instead of primitives ("primitive obsession")

```csharp
// Primitive obsession: nothing stops arguments being passed in the wrong order
public void Transfer(string fromAccount, string toAccount, decimal amount) { }
Transfer(toAccountId, fromAccountId, amount); // compiles, silently wrong

// A small wrapper type prevents the mix-up at compile time
public readonly record struct AccountId(string Value);
public void Transfer(AccountId from, AccountId to, decimal amount) { }
```

## Application

Apply careful API design to any type or method used across module boundaries or by other developers — public classes in a shared library, service interfaces, domain entities. For small private helper methods used in one place, this level of ceremony is usually unnecessary.

## Common Mistakes

- Exposing setters on every property "for flexibility," allowing invalid combinations of state.
- Using primitive types (`string`, `int`) for concepts with real identity or rules (IDs, money, email addresses), enabling accidental parameter swaps.
- Making implementation details `public` or `protected` without a reason, expanding the contract callers can depend on and locking in details that should be free to change.
- Designing an API around how it's implemented rather than how it will be used, leaking internal structure to the caller.

## Common Interview Questions

### Basic
- What does "make invalid states unrepresentable" mean?
- What is primitive obsession?

### Intermediate
- Why is keeping the public surface of a class minimal important for maintainability?
- How do constructors with validation prevent a whole category of bugs compared to public setters?

### Advanced
- How would you redesign a class with widespread public setters to enforce its invariants without breaking too many existing call sites at once?
- How does good class design reduce the need for defensive `if (x == null)` checks scattered throughout a codebase?

### Follow-up Questions
- Is primitive obsession always worth fixing, or does it depend on scale?
- How does API design relate to the Open/Closed and Liskov Substitution Principles?

### Code Prediction
Given the `Transfer(string fromAccount, string toAccount, decimal amount)` example, what compiler error (if any) occurs if the caller accidentally swaps `fromAccount` and `toAccount`? What changes once `AccountId` wrapper types are introduced?

## Practical Tasks

- Redesign a class with public mutable properties into one where invalid states cannot be constructed.
- Replace two same-typed `string` parameters that are easy to swap with distinct wrapper types.
- Review a public class's surface and reduce it to only what external callers actually need.

## Readiness Criteria

Design constructors and public members that make invalid states hard or impossible to represent, recognize primitive obsession, and keep a type's public surface minimal and intention-revealing.

## References

### Microsoft Learn

- [Framework design guidelines](https://learn.microsoft.com/dotnet/standard/design-guidelines/)
- [Choosing between class and struct](https://learn.microsoft.com/dotnet/standard/design-guidelines/choosing-between-class-and-struct)
