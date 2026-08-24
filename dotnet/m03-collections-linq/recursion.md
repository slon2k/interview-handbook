# Recursion

## Definition

Recursion solves a problem by calling the same method on a smaller input until a base case is reached.

```csharp
static int Sum(int[] values, int index)
{
    if (index == values.Length) return 0;
    return values[index] + Sum(values, index + 1);
}
```

Every recursive solution needs a valid base case, progress toward that case, and correct combination of the recursive result.

## Alternatives & Trade-offs

Recursion can express trees, divide-and-conquer, and backtracking clearly. Iteration usually avoids call-stack growth and may be preferable for deep or adversarial input. An explicit stack can preserve recursive logic without using the call stack.

## How It Works

Each call creates a stack frame containing parameters and local state. A recursion depth of $d$ uses approximately $O(d)$ call-stack space. Tail-call optimization should not be assumed in ordinary C# code.

## Application

- Traverse trees and nested structures.
- Divide problems into independent subproblems.
- Explore backtracking choices.
- Implement recursive definitions where the depth is controlled.

## Common Mistakes

- Omitting the base case.
- Making no progress toward termination.
- Recomputing overlapping subproblems without memoization.
- Ignoring stack-overflow risk.
- Claiming constant space while recursive frames grow with depth.

## Common Interview Questions

### Basic

- What is a base case?
- What is the call stack?
- What makes recursion terminate?

### Intermediate

- How do recursive time and space complexity differ?
- When should recursion be replaced by iteration?
- What is memoization?

### Advanced

- How do you derive a recurrence relation?
- When does divide-and-conquer reduce complexity?
- How can memoization change exponential recursion to polynomial time?
- How would you convert a recursive traversal to an explicit stack?
- What stack-depth and input-validation risks exist in production code?

### Follow-up Questions

- What happens if the base case is unreachable?
- Does C# guarantee tail-call elimination?
- How can you test recursive edge cases?

### Code Prediction

What is the result?

```csharp
static int CountDown(int value)
{
    if (value == 0) return 0;
    return 1 + CountDown(value - 1);
}
```

## Practical Tasks

- Implement factorial and Fibonacci with iterative and recursive versions.
- Add memoization to a recursive solution with overlapping subproblems.
- Convert a recursive traversal to an explicit stack.

## Readiness Criteria

Identify base cases and progress, calculate time and stack-space complexity, recognize repeated work, and choose recursion or iteration deliberately.

## References

### Microsoft Learn

- [Methods](https://learn.microsoft.com/dotnet/csharp/programming-guide/classes-and-structs/methods)
- [Stack<T> class](https://learn.microsoft.com/dotnet/api/system.collections.generic.stack-1)
