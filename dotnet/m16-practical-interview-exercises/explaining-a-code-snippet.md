# Explaining a Code Snippet

## What This Assesses

Given an unfamiliar or deliberately tricky snippet, can you explain precisely what it does and why — including predicting its output? This is the most concentrated form of the "code prediction" question style used throughout every module's topic files, and it specifically probes depth over memorized syntax.

## Format and Time Expectations

A snippet of 5-20 lines, often followed by "what does this print?" or "what's wrong with this?" — narrate your reasoning step by step rather than jumping straight to an answer.

## Exercise 1: Value vs. Reference Semantics

**Problem:** What does this print?

```csharp
struct Point { public int X; }
class Container { public Point P; }

void Modify(Point p) { p.X = 99; }
void ModifyContainer(Container c) { c.P.X = 99; }

var point = new Point { X = 1 };
Modify(point);
Console.WriteLine(point.X); // ?

var container = new Container { P = new Point { X = 1 } };
ModifyContainer(container);
Console.WriteLine(container.P.X); // ?
```

**What a strong answer demonstrates:** Correctly explaining that `Point` is a struct (value type, Module 2) — `Modify(point)` receives a *copy*, so `point.X` stays `1`. `Container` is a class (reference type) — `ModifyContainer(container)` receives a reference to the *same* object, so mutating `c.P.X` does affect `container.P.X`, printing `99`.

**Common mistakes:** Assuming everything passed to a method behaves the same way regardless of whether it's a struct or class, missing the value/reference distinction entirely.

## Exercise 2: Static Field and Object Identity

**Problem:** What does this print?

```csharp
public class Counter
{
    private static int _count = 0;
    public int Id;
    public Counter() { Id = ++_count; }
}

var a = new Counter();
var b = new Counter();
Console.WriteLine($"{a.Id}, {b.Id}, {Counter._count}"); // conceptually — assume access for illustration
```

**What a strong answer demonstrates:** Explaining that `_count` is shared across *every* instance (Module 2's static-members content) — each `new Counter()` increments the same underlying field, so `a.Id = 1`, `b.Id = 2`, and the static count is `2`.

**Common mistakes:** Assuming each instance gets its own independent copy of a static field, the same misunderstanding that causes real bugs with shared mutable static state.

## Exercise 3: Boxing and Equality

**Problem:** What does this print, and why?

```csharp
object a = 5;
object b = 5;
Console.WriteLine(a == b);        // ?
Console.WriteLine(a.Equals(b));   // ?
```

**What a strong answer demonstrates:** Explaining boxing (Module 2/13) — both `5`s are boxed into separate heap objects. `==` on `object` references compares reference identity by default (not overridden for boxed `int`), so it's `false`. `.Equals()` on `int` (even boxed) uses value equality, so it's `true`. This distinction — same underlying value, different answer depending on which comparison mechanism is used — is exactly the kind of subtlety this exercise type is designed to surface.

**Common mistakes:** Assuming `==` and `.Equals()` always agree, or assuming boxed integers compare by reference for both.

## Exercise 4: LINQ Deferred Execution with a Twist

**Problem:** What does this print?

```csharp
var list = new List<int> { 1, 2, 3 };
var doubled = list.Select(x => x * 2);
Console.WriteLine(string.Join(",", doubled)); // first enumeration
list.Clear();
Console.WriteLine(string.Join(",", doubled)); // second enumeration
```

**What a strong answer demonstrates:** Correctly explaining that `doubled` is a lazy query, re-evaluated against `list`'s *current* contents every time it's enumerated (Module 3) — the first line prints `2,4,6`; after `list.Clear()`, the second enumeration re-runs against the now-empty list and prints an empty string, not the previously-seen `2,4,6`.

**Common mistakes:** Assuming the result of `Select` is "computed once" the first time it's used, missing that each enumeration re-executes the query from scratch.

## Readiness Criteria

Predict output correctly for snippets involving value/reference semantics, static state, boxing/equality, and deferred execution, and explain the underlying mechanism precisely rather than guessing at the answer.

## References

- [Value and reference semantics (Module 2)](../m02-csharp-language/value-and-reference-semantics.md)
- [Static members and static classes (Module 2)](../m02-csharp-language/static-members-and-static-classes.md)
- [Equality and hashing (Module 2)](../m02-csharp-language/equality-and-hashing.md)
- [Deferred execution and materialization (Module 3)](../m03-collections-linq/deferred-execution-and-materialization.md)
