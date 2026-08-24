# SOLID Principles

## Definition

SOLID is a group of design principles: Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, and Dependency Inversion.

## Alternatives & Trade-offs

SOLID helps reason about change and substitution; it is guidance, not a rule to maximize abstractions or class count.

## How It Works

- **SRP:** one reason to change.
- **OCP:** extend behavior without modifying stable code where practical.
- **LSP:** subtypes remain substitutable.
- **ISP:** clients depend on focused interfaces.
- **DIP:** high-level policy does not depend directly on low-level details.

## Application

Use SOLID during design and code review to identify responsibility, substitution, and dependency problems.

## Common Mistakes

- Treating SRP as one method per class.
- Using OCP to justify speculative plugin systems.
- Applying LSP only to syntax rather than behavior.
- Creating interfaces with no real client boundary.

## Common Interview Questions

### Basic
- What does SOLID stand for?
- What is the Dependency Inversion Principle?

### Intermediate
- Give an example of an LSP violation.
- How does interface segregation help clients?

### Advanced
- How do SOLID principles conflict with simplicity?
- How would you detect a false abstraction introduced for OCP?
- How do architecture boundaries support DIP?

### Follow-up Questions
- Are SOLID principles always applicable?
- How are DIP and dependency injection related?

### Code Prediction
Which change would violate LSP: a subtype rejecting valid base inputs or changing implementation details?

## Practical Tasks

- Review a class against each SOLID principle.
- Refactor one real violation without adding speculative abstractions.

## Readiness Criteria

Explain each principle, identify practical violations, and balance design quality against simplicity.

## References

### Microsoft Learn

- [Interfaces](https://learn.microsoft.com/dotnet/csharp/fundamentals/types/interfaces)
- [Dependency injection](https://learn.microsoft.com/dotnet/core/extensions/dependency-injection)
