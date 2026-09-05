# Modules and Error Handling

## Definition

ES modules give each file its own scope and explicit imports and exports. JavaScript errors interrupt normal execution until handled; `try`/`catch`/`finally` and rejected promises provide boundaries for reporting or recovering from failures.

## How It Works

- Use named exports for a module's explicit public surface and default exports only when one primary export is genuinely clearer.
- Imports are statically declared, read-only bindings. Modules execute once and their exports can be shared by importers.
- Throw `Error` objects, ideally with a meaningful message and an appropriate built-in or domain-specific type.
- `try`/`catch` handles synchronous exceptions and errors thrown by awaited promises inside the `try` block.
- `finally` runs whether work succeeds or fails; use it for cleanup, not for replacing the actual failure.
- Validate unknown caught values before accessing properties; JavaScript can throw values that are not `Error` objects.

## Application

Keep modules focused with clear dependencies and exports. Catch errors at a boundary that can decide how to recover or present the failure; do not silently swallow errors simply to keep a UI moving.

## Common Mistakes

- Making every helper a default export, obscuring its public name.
- Catching an error and continuing with invalid or invented data.
- Throwing strings rather than `Error` instances.
- Treating a `catch` around promise creation as handling a later rejection without `await` or `.catch`.

## Common Interview Questions

### Foundation

- What problem do ES modules solve?
- What is the purpose of `finally`?

### Intermediate

- Where should a UI application catch and present a failed operation?
- Why should code not assume a caught value is an `Error`?

### Advanced and Follow-up

- How would you organise modules to avoid a circular dependency between UI and data logic?

### Code Prediction

Predict whether a `finally` block runs when a function returns from its `try` block and when it throws.

## Practical Tasks

- Split a mixed utility file into focused modules with explicit exports.
- Add an error boundary around an asynchronous operation without hiding the error from diagnostics.

## Readiness Criteria

You can define a clear module surface and handle failures at a boundary that can make an informed recovery decision.

## References

- [MDN: JavaScript modules](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules)
- [MDN: try...catch](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/try...catch)