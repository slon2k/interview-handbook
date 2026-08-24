# Refactoring Techniques

## Definition

Refactoring changes internal structure without changing externally observable behavior.

## Alternatives & Trade-offs

Small refactorings reduce risk and are easier to review. Large rewrites may be justified by an unworkable design but require stronger characterization and rollout plans.

## How It Works

A safe refactoring uses tests, a small step, compilation, and review. Common techniques include extract method, extract class, rename, introduce parameter object, and replace conditional with polymorphism.

## Application

Refactor when repeated change, defects, unclear responsibilities, or coupling create measurable maintenance cost.

## Common Mistakes

- Changing behavior accidentally.
- Refactoring without tests or observability.
- Mixing broad formatting with semantic changes.
- Rewriting instead of improving incrementally.

## Common Interview Questions

### Basic
- What is refactoring?
- Does refactoring change behavior?

### Intermediate
- What makes a refactoring safe?
- When would you extract a class?

### Advanced
- How do you refactor code with no tests?
- How do you preserve public API compatibility?
- How can refactoring affect performance?

### Follow-up Questions
- Should refactoring happen in a separate commit?
- When is a rewrite justified?

### Code Prediction
Which change is behavior-preserving: renaming a private method or changing a validation rule?

## Practical Tasks

- Add characterization tests before refactoring a legacy method.
- Apply two small refactorings and review the resulting dependency graph.

## Readiness Criteria

Explain behavior preservation, test safety, incremental techniques, compatibility, and when a rewrite is or is not justified.

## References

### Microsoft Learn

- [Refactoring](https://learn.microsoft.com/visualstudio/ide/refactoring-in-visual-studio)
