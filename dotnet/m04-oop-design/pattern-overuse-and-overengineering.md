# Pattern Overuse and Overengineering

## Definition

Overengineering adds abstraction, configuration, or flexibility beyond current requirements. Pattern overuse applies a named solution where a simpler design would be clearer.

## Alternatives & Trade-offs

A direct implementation minimizes complexity. A pattern earns its cost when it addresses a recurring change or collaboration problem.

## How It Works

Every abstraction adds concepts, indirection, and maintenance cost. Evaluate the expected change, current evidence, and reversibility before introducing a framework or pattern.

## Application

Prefer simple composition first; introduce patterns when variation, testing, or extension points are real and understood.

Singleton is a common example of a pattern that can become harmful: it introduces global lifetime, hidden dependencies, and shared mutable state. Prefer explicit dependencies and a DI-managed lifetime when one process-wide instance is genuinely required.

## Common Mistakes

- Building generic frameworks for one use case.
- Adding factories that only call constructors.
- Treating pattern names as design quality.

## Common Interview Questions

### Basic
- What is overengineering?
- When is a design pattern useful?

### Intermediate
- How do you decide whether abstraction is worth it?
- What is the cost of indirection?

### Advanced
- How do you preserve future flexibility without speculative design?
- How can patterns create accidental coupling?
- How would you simplify an over-abstracted design safely?

### Follow-up Questions
- Is a simple design always better?
- What evidence justifies a pattern?

### Code Prediction
Which design has fewer moving parts while satisfying one fixed behavior?

## Practical Tasks

- Simplify a pattern-heavy example into direct composition.
- Identify the requirement that would justify restoring an abstraction.

## Readiness Criteria

Evaluate patterns by problem, change pressure, complexity, and evidence rather than by vocabulary or fashion.

## References

### Microsoft Learn

- [Design guidelines](https://learn.microsoft.com/dotnet/standard/design-guidelines/)
