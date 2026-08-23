# Variables, Expressions and Control Flow

## Definition

Variables are named storage locations that hold values during program execution.

Every variable has:

- A name
- A type
- A scope
- A lifetime

Example:

```csharp
int count = 10;
string message = "Hello";
```

An expression is a piece of code that produces a value.

Examples:

```csharp
10 + 20
count > 0
user.Name
GetTotal()
```

Control flow determines the order in which code executes.

Common control-flow constructs include:

- `if` / `else`
- `switch`
- `switch` expressions
- `for`
- `foreach`
- `while`
- `do-while`
- `break`
- `continue`
- `return`

This topic forms the foundation of all C# programming and is required for understanding every other language feature.

## Alternatives & Trade-offs

### `if`/`else` vs `switch`

Use `if`/`else` when conditions are complex or involve multiple unrelated predicates.

```csharp
if (age >= 18 && hasLicense)
{
    ...
}
```

Use `switch` when selecting behavior based on a value or pattern.

```csharp
switch (status)
{
    case OrderStatus.New:
        ...
        break;
}
```

Switch expressions are often more concise and readable.

```csharp
var message = status switch
{
    OrderStatus.New => "New",
    OrderStatus.Completed => "Completed",
    _ => "Unknown"
};
```

### `for` vs `foreach`

Use `foreach` when iterating through a collection without needing the index.

```csharp
foreach (var item in items)
{
    Console.WriteLine(item);
}
```

Use `for` when the index is required or iteration logic is more complex.

```csharp
for (int i = 0; i < items.Count; i++)
{
    Console.WriteLine(items[i]);
}
```

### Statement vs Expression

Statements perform actions.

```csharp
if (isValid)
{
    Save();
}
```

Expressions produce values.

```csharp
isValid && isEnabled
```

Modern C# increasingly favors expression-based syntax because it often improves readability.

## How It Works

### Variable Declaration

Variables can be declared explicitly:

```csharp
int count = 5;
```

Or using type inference:

```csharp
var count = 5;
```

The compiler still determines a concrete type.

```csharp
var count = 5; // int
```

### Scope

A variable is only accessible within its scope.

```csharp
if (true)
{
    int number = 10;
}

// number is not accessible here
```

### Definite Assignment

Local variables must be assigned before use.

```csharp
int value;
Console.WriteLine(value);
```

This does not compile.

```text
Use of unassigned local variable
```

### Expression Evaluation

Expressions are evaluated according to operator precedence.

```csharp
int result = 2 + 3 * 4;
```

Result:

```text
14
```

because multiplication is evaluated before addition.

Parentheses can change evaluation order.

```csharp
int result = (2 + 3) * 4;
```

Result:

```text
20
```

### Short-Circuit Evaluation

Logical operators `&&` and `||` may skip evaluating part of an expression.

```csharp
if (user != null && user.IsActive)
{
    ...
}
```

If `user` is `null`, the second condition is never evaluated.

### Control Flow

Control-flow statements determine which code path executes.

```csharp
if (temperature > 25)
{
    Console.WriteLine("Warm");
}
else
{
    Console.WriteLine("Cold");
}
```

Loops repeatedly execute code until a condition changes.

```csharp
while (count > 0)
{
    count--;
}
```

## Application

Variables, expressions, and control flow appear in virtually every application.

Typical use cases:

### Validation

```csharp
if (string.IsNullOrWhiteSpace(email))
{
    throw new ArgumentException("Email is required");
}
```

### Business Rules

```csharp
if (order.Total >= 100)
{
    ApplyDiscount();
}
```

### Data Processing

```csharp
foreach (var order in orders)
{
    Process(order);
}
```

### State Transitions

```csharp
var nextState = currentState switch
{
    OrderState.New => OrderState.Processing,
    OrderState.Processing => OrderState.Completed,
    _ => currentState
};
```

## Common Mistakes

### Using Variables Before Initialization

```csharp
int count;
Console.WriteLine(count);
```

Local variables must be assigned before use.

### Confusing Assignment and Equality

Incorrect:

```csharp
if (isValid = true)
{
}
```

Correct:

```csharp
if (isValid == true)
{
}
```

or simply:

```csharp
if (isValid)
{
}
```

### Off-by-One Errors

Incorrect:

```csharp
for (int i = 0; i <= items.Count; i++)
{
}
```

This attempts to access an index beyond the collection bounds.

Correct:

```csharp
for (int i = 0; i < items.Count; i++)
{
}
```

### Deeply Nested Conditions

Hard to maintain:

```csharp
if (...)
{
    if (...)
    {
        if (...)
        {
        }
    }
}
```

Consider guard clauses instead.

### Ignoring Short-Circuit Behavior

Candidates often know `&&` and `||` but forget that later conditions may never execute.

## Common Interview Questions

### Basic

- What is the difference between a variable and a value?
- What is an expression?
- What is variable scope?
- What is variable lifetime?
- What is the difference between a statement and an expression?

### Intermediate

- What is type inference?
- What does `var` actually mean?
- What is short-circuit evaluation?
- What is the difference between `for` and `foreach`?
- When would you choose `switch` instead of `if`?

### Follow-up Questions

- Does `var` make C# dynamically typed?
- Why must local variables be initialized before use?
- What happens if a `break` statement is omitted in a traditional switch?
- Can a `foreach` loop modify the collection it is iterating over?
- Why were switch expressions introduced?

### Code Prediction

What is the output?

```csharp
int x = 5;

if (x > 3 || ThrowException())
{
    Console.WriteLine("True");
}
```

Would `ThrowException()` execute? Why?

---

What is the output?

```csharp
int result = 2 + 3 * 4;
Console.WriteLine(result);
```

Explain the result.

---

What is the output?

```csharp
for (int i = 0; i < 3; i++)
{
    Console.Write(i);
}
```

## Practical Tasks

### Refactoring

Convert:

```csharp
if (status == OrderStatus.New)
{
    return "New";
}
else if (status == OrderStatus.Completed)
{
    return "Completed";
}
else
{
    return "Unknown";
}
```

into a switch expression.

### Debugging

Find and fix the bug:

```csharp
for (int i = 0; i <= items.Count; i++)
{
    Console.WriteLine(items[i]);
}
```

### Code Review

Review:

```csharp
if (user != null)
{
    if (user.IsActive)
    {
        if (user.HasAccess)
        {
            Process(user);
        }
    }
}
```

Refactor using guard clauses.

### Implementation

Implement a method that categorizes a score:

```text
90-100 => Excellent
70-89 => Good
50-69 => Pass
0-49 => Fail
```

Use both:

- `if`/`else`
- `switch` expression

## Readiness Criteria

You should be able to:

- Explain variables, expressions, and control flow in your own words.
- Describe scope and lifetime.
- Explain type inference and `var`.
- Predict expression evaluation and operator precedence.
- Explain short-circuit evaluation.
- Choose between `if`, `switch`, `for`, and `foreach`.
- Identify common control-flow bugs.
- Refactor complex conditional logic into simpler forms.
- Confidently answer common interview follow-up questions.

## References

### Microsoft Learn

- Variables
- Expressions
- Selection statements (`if`, `switch`)
- Iteration statements (`for`, `foreach`, `while`, `do`)
- C# operators and expressions

### Additional Reading

- C# Language Specification
- C# in Depth — Jon Skeet
- CLR via C# — Jeffrey Richter
