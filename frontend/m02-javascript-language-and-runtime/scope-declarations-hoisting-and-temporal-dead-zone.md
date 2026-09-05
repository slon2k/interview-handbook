# Scope, Declarations, Hoisting, and the Temporal Dead Zone

## Definition

Scope determines where a binding can be used. `let` and `const` are block-scoped; `var` is function-scoped. Hoisting describes how declarations are processed before a scope's code executes. The temporal dead zone is the period before a `let` or `const` declaration is initialized.

## How It Works

- Prefer `const`; use `let` only for bindings that must be reassigned. Avoid `var` in modern code.
- `let` and `const` exist from the start of their block but accessing them before their declaration throws a `ReferenceError`.
- Function declarations are callable before their textual declaration in the same scope.
- `var` declarations are initialized to `undefined`, which produces a different failure mode and can leak through blocks.
- Each iteration of a `for (let ...)` loop has a distinct binding, unlike the common `var` closure pitfall.

## Application

Keep bindings in the narrowest scope that expresses their lifetime. This prevents accidental reuse, makes data flow legible, and avoids event handlers observing an unexpected loop variable.

## Common Mistakes

- Accessing a `let` binding before initialization and calling it “undefined.”
- Using `var` in a loop whose callbacks run later.
- Hoisting declarations to a broad function scope instead of extracting a small function.

## Common Interview Questions

### Foundation

- How do `var`, `let`, and `const` differ?
- What is the temporal dead zone?

### Intermediate

- Why do callbacks created in a `var` loop often see the final index?
- What is hoisted for a function declaration versus a `const` function expression?

### Advanced and Follow-up

- How does block scope reduce bugs in asynchronous UI code?

### Code Prediction

Predict the result of reading a `var` before its assignment and reading a `let` before its declaration.

## Practical Tasks

- Fix a loop that registers handlers with the wrong index.
- Refactor a function that uses broad mutable bindings into narrow `const` and `let` scopes.

## Readiness Criteria

You can explain binding lifetime, hoisting, and the temporal dead zone and use declarations predictably.

## References

- [MDN: let](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/let)
- [MDN: Hoisting](https://developer.mozilla.org/docs/Glossary/Hoisting)