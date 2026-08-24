# Code Smells

## Definition

A code smell is a surface indicator in source code that usually signals a deeper design problem — not a bug itself, but a sign that refactoring is likely worthwhile. Common examples: long methods, large classes, long parameter lists, duplicated code, feature envy, and shotgun surgery.

```csharp
// Long parameter list + primitive obsession — two smells at once
public void CreateUser(string firstName, string lastName, string email, string phone,
    string street, string city, string zip, DateTime birthDate, bool isAdmin) { }
```

## Alternatives & Trade-offs

Recognizing smells early is cheap; ignoring them lets small frictions compound into a codebase that's expensive to change. Not every smell needs fixing immediately — a "God class" in code that's about to be deleted isn't worth refactoring, and treating every smell as an emergency can itself become a form of over-engineering. The judgment call is whether the smelly code is on a path of continued change.

## How It Works

### Long method

```csharp
public void ProcessOrder(Order order)
{
    // 80 lines mixing validation, pricing, inventory checks, payment, notification...
}
```
Smell: doing too much in one place, hard to test any single piece in isolation. Fix: extract cohesive steps into named methods or collaborator classes.

### Feature envy

```csharp
public class InvoicePrinter
{
    public string Print(Order order) =>
        $"{order.Customer.Name}, {order.Customer.Address.Street}, {order.Customer.Address.City}, total: {order.Items.Sum(i => i.Price)}";
}
```
`InvoicePrinter` reaches deep into `order.Customer.Address` and `order.Items` — it's more interested in `Order`'s data than its own. Fix: move the summarizing logic onto `Order` itself, or introduce a dedicated view/DTO built by `Order`.

### Shotgun surgery

A single conceptual change (e.g., adding a new `OrderStatus`) that requires editing the same `switch` statement in six different unrelated classes. Smell: the concept isn't centralized. Fix: consolidate the `switch` behavior behind polymorphism (see Strategy) or a single owning type.

### Duplicated code

```csharp
// In OrderService
if (email.Contains('@') && email.Length <= 254) { }

// In UserService, independently duplicated
if (email.Contains('@') && email.Length <= 254) { }
```
Fix: extract to a single validator — but only once a second real use case confirms the rule is actually shared (see DRY/YAGNI).

## Application

Use smells as a checklist during code review and before extending a piece of code: is this method doing one thing, is this class cohesive, would a single conceptual change require touching many files? Treat smells as prompts to investigate, not automatic verdicts.

## Common Mistakes

- Refactoring purely because a smell is "textbook present," without checking whether the code is stable and unlikely to change (low payoff for the risk of touching it).
- Fixing a smell locally (e.g., extracting a method) without addressing the underlying design issue it was pointing at (e.g., an SRP violation).
- Treating every long method as automatically bad — a long method that is a single linear sequence with no real branching or reuse concern may not need splitting.
- Confusing "duplicated code" with "similar-looking code that represents different business rules" (see DRY/KISS/YAGNI).

## Common Interview Questions

### Basic
- What is a code smell, and how does it differ from a bug?
- Name three common code smells.

### Intermediate
- What is feature envy, and what design change typically fixes it?
- What is shotgun surgery, and what does it usually indicate about how a concept is modeled?

### Advanced
- How do you prioritize which smells to fix first in a large, imperfect legacy codebase?
- How does recognizing a smell connect back to the SOLID principle it's actually violating?

### Follow-up Questions
- Is a long method always a problem?
- Can code have zero "textbook" smells and still be poorly designed?

### Code Prediction
Given the `InvoicePrinter` feature-envy example, if `Order`'s internal structure changes (e.g., `Customer.Address` becomes `Customer.BillingAddress`), how many places likely need to change? What would change about that if the formatting logic lived on `Order` itself?

## Practical Tasks

- Identify feature envy in a given class and refactor it by relocating the envious logic.
- Refactor a long method into named, single-purpose steps without changing its behavior.
- Trace a shotgun-surgery scenario (adding a new enum case requiring edits in six files) and consolidate it behind polymorphism.

## Readiness Criteria

Recognize common code smells by name and by pattern in real code, connect each smell to the design principle it violates, and judge when a smell is worth fixing now versus leaving alone.

## References

### Microsoft Learn

- [Common architectural principles](https://learn.microsoft.com/dotnet/architecture/modern-web-apps-azure/architectural-principles)
- [Code metrics values](https://learn.microsoft.com/visualstudio/code-quality/code-metrics-values)
