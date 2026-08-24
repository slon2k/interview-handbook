# Encapsulation

## Definition

Encapsulation bundles data with the behavior that operates on it, and restricts direct access to internal state so an object can enforce its own invariants. It is the mechanism that makes "an object cannot be put into an invalid state from the outside" possible.

```csharp
public sealed class BankAccount
{
    public decimal Balance { get; private set; }

    public void Deposit(decimal amount)
    {
        if (amount <= 0) throw new ArgumentOutOfRangeException(nameof(amount));
        Balance += amount;
    }
}
```

## Alternatives & Trade-offs

Compared to exposing public mutable fields, encapsulation costs a small amount of boilerplate (properties, methods) in exchange for guaranteed invariants and a stable public contract that can change its internal representation without breaking callers. Under-encapsulating (public setters everywhere) trades safety for convenience; over-encapsulating (wrapping every trivial field in defensive logic) can add needless ceremony to simple data holders — a plain DTO doesn't need the same rigor as a domain entity.

## How It Works

### Broken encapsulation

```csharp
public class BankAccount
{
    public decimal Balance; // any code can set this to anything, including a negative value
}

account.Balance = -500; // compiles, silently corrupts state
```

### Encapsulation via properties and methods

```csharp
public sealed class BankAccount
{
    private decimal _balance;
    public decimal Balance => _balance; // read-only from outside

    public void Withdraw(decimal amount)
    {
        if (amount <= 0) throw new ArgumentOutOfRangeException(nameof(amount));
        if (amount > _balance) throw new InvalidOperationException("Insufficient funds");
        _balance -= amount;
    }
}
```

The only way to change `_balance` is through `Withdraw`/`Deposit`, so the invariant "balance never negative" cannot be bypassed.

### Encapsulation with records

```csharp
public sealed record Money(decimal Amount, string Currency)
{
    public Money
    {
        if (Amount < 0) throw new ArgumentOutOfRangeException(nameof(Amount));
    }
}
```

The primary constructor's validation body runs on every construction, so an invalid `Money` cannot be created at all.

## Application

Apply strong encapsulation to domain entities and value objects where invalid state is a real risk (money, dates, identifiers, business rules). Simple, short-lived DTOs used purely for data transfer across a boundary typically don't need the same defensive rigor.

## Common Mistakes

- Exposing a public setter "just in case," defeating the invariant the class exists to protect.
- Encapsulating data but leaking mutable internal collections (`public List<Item> Items => _items;` lets callers `Clear()` it directly).
- Validating in a public method but leaving another public method or a public constructor as a bypass.
- Confusing encapsulation with access modifiers alone — `private` fields with public getters and setters for every field is not meaningfully more encapsulated than public fields.

## Common Interview Questions

### Basic
- What is encapsulation, and how does it differ from just using `private` fields?
- What is an invariant?

### Intermediate
- Why is exposing a mutable `List<T>` property a common encapsulation leak?
- How do records help enforce invariants at construction time?

### Advanced
- How do you encapsulate a collection so it can be read but not externally mutated?
- How does encapsulation support the Open/Closed Principle by allowing internal representation to change freely?

### Follow-up Questions
- Is a class with all-public auto-properties still "encapsulated"?
- Should validation live in the constructor, in a factory method, or in setters?

### Code Prediction
```csharp
public class Order
{
    public List<string> Items { get; } = new();
}

var order = new Order();
order.Items.Clear();
```
Does this compile? What does it reveal about the encapsulation of `Order`, even though `Items` has no public setter?

## Practical Tasks

- Refactor `BankAccount` with a public `Balance` field into a properly encapsulated version with `Deposit`/`Withdraw`.
- Fix the `Order.Items` leak above so callers can enumerate items but not clear or mutate the underlying list.
- Design a `DateRange` type that cannot be constructed with `End` earlier than `Start`.

## Readiness Criteria

Explain encapsulation in terms of invariants (not just access modifiers), identify common leaks (mutable collection exposure, bypassable validation), and design a type that cannot be put into an invalid state.

## References

### Microsoft Learn

- [Object-oriented programming concepts](https://learn.microsoft.com/dotnet/csharp/fundamentals/tutorials/oop)
- [Properties](https://learn.microsoft.com/dotnet/csharp/programming-guide/classes-and-structs/properties)
- [Records](https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/record)
