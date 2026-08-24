# Code Smells

## Definition

A code smell is a symptom of design or maintenance difficulty, not necessarily a defect. Examples include long methods, large classes, duplication, feature envy, and shotgun changes.

## Alternatives & Trade-offs

A smell is a signal to investigate, not an automatic refactoring command. Some duplication or complexity may be justified by domain requirements.

## How It Works

Smells indicate excessive responsibility, coupling, unclear intent, or unstable boundaries. Tests and measurements help determine whether a change improves the design.

## Common Examples

### Long Method

A method that validates input, queries storage, applies business rules, formats output, and sends notifications has too many reasons to change.

```csharp
public InvoiceResult Process(InvoiceRequest request)
{
	Validate(request);
	Customer customer = database.LoadCustomer(request.CustomerId);
	decimal total = CalculateTotal(request, customer);
	Invoice invoice = formatter.Format(total);
	email.Send(invoice);
	return new InvoiceResult(invoice);
}
```

Extract focused collaborators or methods only when the boundaries improve understanding and change isolation.

### Large Class

A class that owns persistence, authorization, pricing, reporting, and email has low cohesion. Separate responsibilities by reason to change, not by arbitrary line count.

### Duplicate Knowledge

Repeating the same tax rule in checkout and reporting can cause inconsistent behavior. Centralize the rule when both locations represent the same business knowledge. Similar-looking code with independent rules should not be merged automatically.

### Primitive Obsession

Passing a raw string for an email address, currency, or status can allow invalid values everywhere. A value object can enforce the invariant once.

### Feature Envy

A method that reads many fields from another object and performs that object's business logic may belong on the object it relies on.

### Shotgun Changes

If one small requirement requires edits across many unrelated classes, the behavior may be scattered across poor boundaries. A focused service or policy can localize the change.

### Speculative Generality

Unused extension points, generic abstractions, and configuration options add complexity without a current requirement. Remove them until evidence justifies the flexibility.

## Application

Use smells during code review and maintenance planning to prioritize changes that reduce defect or change cost.

## Common Mistakes

- Refactoring without characterization tests.
- Treating style preferences as design defects.
- Applying a pattern to hide a smell.

## Common Interview Questions

### Basic
- What is a code smell?
- Name common code smells.

### Intermediate
- How do you decide whether to refactor?
- Why are long methods risky?

### Advanced
- How do you distinguish a smell from a domain-driven complexity?
- How can a refactoring worsen coupling?
- How do tests and metrics guide smell remediation?

### Follow-up Questions
- Is a code smell always a bug?
- What should be done before a risky refactoring?

### Code Prediction
Which design makes a change require edits across unrelated classes?

## Practical Tasks

- Identify three smells in a legacy class.
- Rank them by risk and propose the smallest safe change.
- Refactor the long-method example while preserving its observable behavior.
- Decide whether two similar code fragments represent duplicated knowledge or independent rules.

## Readiness Criteria

Recognize common smells, explain their risks, and choose evidence-based refactorings without treating every preference as a defect.

## References

### Microsoft Learn

- [Design guidelines](https://learn.microsoft.com/dotnet/standard/design-guidelines/)
