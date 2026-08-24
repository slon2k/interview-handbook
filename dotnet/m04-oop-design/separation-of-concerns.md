# Separation of Concerns

## Definition

Separation of concerns assigns different responsibilities to distinct components so each can change and be tested independently.

## Alternatives & Trade-offs

Separation improves maintainability but excessive layers add ceremony. Separate concerns where they have different reasons to change.

## How It Works

A component should coordinate a focused responsibility and delegate persistence, transport, validation, and policy concerns to appropriate collaborators.

## Application

Separate domain rules from HTTP, database, serialization, and infrastructure details.

## Common Mistakes

- Putting SQL and business rules in controllers.
- Creating layers with no meaningful responsibility.
- Duplicating validation across every layer.

## Common Interview Questions

### Basic
- What is separation of concerns?
- Why should controllers be thin?

### Intermediate
- How do layers reduce change impact?
- Where should business validation live?

### Advanced
- How can boundaries prevent infrastructure leakage?
- When does layering become accidental complexity?
- How do module boundaries support independent testing?

### Follow-up Questions
- Is one class per concern enough?
- What is the difference between validation and business rules?

### Code Prediction
Which component should own a domain invariant independent of HTTP?

## Practical Tasks

- Refactor a controller containing persistence and business logic.
- Map responsibilities across a small application.

## Readiness Criteria

Assign responsibilities by reason to change, avoid infrastructure leakage, and recognize both missing and excessive separation.

## References

### Microsoft Learn

- [Dependency injection guidelines](https://learn.microsoft.com/dotnet/core/extensions/dependency-injection-guidelines)
