# DRY, KISS, and YAGNI

## Definition

DRY avoids duplicated knowledge, KISS favors understandable solutions, and YAGNI discourages speculative features.

## Alternatives & Trade-offs

Removing every repeated line can create premature abstractions. Duplication is sometimes cheaper than coupling two concepts that may evolve differently.

## How It Works

Apply these principles to design decisions: identify whether duplication is accidental or meaningful, prefer the simplest correct implementation, and defer unrequested flexibility.

## Application

Use them during refactoring and review, especially when deciding whether to introduce a shared helper or framework.

## Common Mistakes

- Treating textual duplication as duplicated knowledge.
- Building extension points before a use case exists.
- Choosing clever code over readable code.

## Common Interview Questions

### Basic
- What does DRY mean?
- What does YAGNI mean?

### Intermediate
- When is duplication acceptable?
- How does KISS affect API design?

### Advanced
- How do you distinguish harmful duplication from independent concepts?
- How can DRY increase coupling?
- How would you remove speculative complexity safely?

### Follow-up Questions
- Is duplication always a code smell?
- Can a simple solution be poorly designed?

### Code Prediction
Which change adds unnecessary complexity without a current requirement?

## Practical Tasks

- Review duplicated code and decide whether to consolidate it.
- Remove a speculative abstraction while preserving behavior.

## Readiness Criteria

Use DRY, KISS, and YAGNI as trade-off tools rather than absolute rules.

## References

### Microsoft Learn

- [Design guidelines](https://learn.microsoft.com/dotnet/standard/design-guidelines/)
