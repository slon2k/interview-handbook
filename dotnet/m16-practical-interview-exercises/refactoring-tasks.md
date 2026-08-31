# Refactoring Tasks

## What This Assesses

Refactoring exercises check whether you can improve code's structure without changing its behavior — recognizing code smells (Module 4), applying the right refactoring technique, and doing so incrementally and safely rather than rewriting wholesale. This is one of the most realistic exercise types, since it mirrors actual day-to-day work far more than a from-scratch coding problem does.

## Format and Time Expectations

Usually a given piece of working-but-flawed code (15-40 lines), with a prompt like "how would you improve this?" — expect to talk through your reasoning as you go, not just silently produce a diff.

## Exercise 1: Long Method with Mixed Concerns

**Problem:** Refactor this method.

```csharp
public decimal ProcessOrder(Order order)
{
    if (order.Items.Count == 0) throw new InvalidOperationException("Empty order");
    decimal total = 0;
    foreach (var item in order.Items) total += item.Price * item.Quantity;
    if (order.CustomerType == "VIP") total *= 0.9m;
    Console.WriteLine($"Processing order for {total}");
    _database.Save(order);
    return total;
}
```

**What a strong answer demonstrates:** Recognizing at least three tangled concerns (validation, calculation, logging, persistence — Module 14's separation-of-concerns content) and extracting each into a focused method or collaborator; replacing the magic string `"VIP"` with an enum or constant; questioning whether this method should be doing I/O (`Console.WriteLine`, `_database.Save`) at all versus being a pure calculation with I/O handled by a caller.

**Common mistakes:** Only renaming variables or reformatting without addressing the actual mixed-responsibility structure, missing what the exercise is really testing (Module 4's cohesion content, applied practically).

## Exercise 2: Type-Checking Chain

**Problem:** Refactor this method to be easier to extend with new shape types.

```csharp
public double CalculateArea(string shapeType, double a, double b)
{
    if (shapeType == "rectangle") return a * b;
    else if (shapeType == "triangle") return 0.5 * a * b;
    else if (shapeType == "circle") return Math.PI * a * a;
    else throw new ArgumentException("Unknown shape");
}
```

**What a strong answer demonstrates:** Recognizing this as a candidate for polymorphism (Module 2/4) — an `IShape` interface with an `Area()` method implemented per shape — and explaining that adding a new shape then requires no change to any existing calculation method (Open/Closed Principle, Module 4).

**Common mistakes:** Just adding a `switch` expression instead of `if/else if` — a cosmetic improvement that doesn't address the actual extensibility problem the exercise is testing.

## Exercise 3: Primitive Obsession and a Risky Parameter List

**Problem:** Refactor this method signature to reduce the risk of a bug from misordered arguments.

```csharp
public void Transfer(string fromAccountId, string toAccountId, decimal amount, string currency) { }
```

**What a strong answer demonstrates:** Introducing small wrapper types (`AccountId`, `Money`) to make an accidental argument swap a compile error instead of a silent runtime bug — Module 4's `api-and-class-design.md` primitive-obsession content, applied to a fresh example.

**Common mistakes:** Only adding XML doc comments describing parameter order, which helps a careful reader but does nothing to prevent the actual bug at compile time.

## Exercise 4: Excessive Nesting

**Problem:** Refactor this method to reduce nesting depth.

```csharp
public void Process(Order order)
{
    if (order != null)
    {
        if (order.Items.Count > 0)
        {
            if (order.Customer != null)
            {
                // actual logic, 4 levels deep
            }
        }
    }
}
```

**What a strong answer demonstrates:** Converting to guard clauses (Module 4) that return/throw early for each invalid condition, leaving the actual logic at the top level of indentation — a direct, well-known refactoring with no behavior change.

**Common mistakes:** Combining all three conditions into one large `if` with `&&`, which does reduce nesting but loses the ability to give a distinct, specific error/behavior for each individual invalid condition.

## Readiness Criteria

Identify the specific code smell driving each refactoring rather than making cosmetic changes, name the refactoring technique or design principle you're applying, and explain why the resulting code is more maintainable — not just different.

## References

- [Code smells (Module 4)](../m04-oop-design/code-smells.md)
- [Refactoring techniques (Module 4)](../m04-oop-design/refactoring-techniques.md)
- [Guard clauses (Module 4)](../m04-oop-design/guard-clauses.md)
- [API and class design (Module 4)](../m04-oop-design/api-and-class-design.md)
