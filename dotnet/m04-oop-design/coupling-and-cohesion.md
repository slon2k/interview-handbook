# Coupling and Cohesion

## Definition

Coupling measures how strongly modules depend on one another. Cohesion measures how closely related the responsibilities inside a module are.

## Alternatives & Trade-offs

Prefer low coupling and high cohesion, but avoid artificial fragmentation. Some coupling is necessary; the goal is stable, intentional dependencies.

## How It Works

Dependencies through concrete types, global state, and knowledge of internal details increase coupling. A focused type with one related responsibility has stronger cohesion.

## Application

Use dependency boundaries, small interfaces, and clear ownership to keep changes local.

## Common Mistakes

- Splitting every method into a class.
- Sharing mutable global state.
- Calling deep object internals instead of a stable operation.

## Common Interview Questions

### Basic
- What is coupling?
- What is cohesion?

### Intermediate
- How do interfaces reduce coupling?
- How can a class become incohesive?

### Advanced
- How do dependency graphs affect change propagation?
- How would you measure architectural coupling?
- When can reducing coupling add harmful indirection?

### Follow-up Questions
- Is low coupling always the goal?
- How does cohesion guide class boundaries?

### Code Prediction
Which design localizes changes when one implementation changes?

## Practical Tasks

- Identify high-coupling dependencies in a service.
- Split an incohesive class along responsibility boundaries.

## Readiness Criteria

Explain coupling and cohesion, identify dependency risks, and justify boundaries without over-fragmenting the design.

## References

### Microsoft Learn

- [Class design guidelines](https://learn.microsoft.com/dotnet/standard/design-guidelines/)
