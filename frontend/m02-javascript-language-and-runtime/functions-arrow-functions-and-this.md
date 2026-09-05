# Functions, Arrow Functions, and `this`

## Definition

Functions are callable objects that can receive arguments, return values, and close over surrounding bindings. Regular functions receive `this` from their call site; arrow functions capture `this` lexically from their enclosing scope.

## How It Works

- Function declarations and expressions can be called with `this` determined by invocation: `object.method()` supplies `object`; a detached call usually does not.
- Arrow functions do not create their own `this`, `arguments`, or constructible prototype.
- Use arrow functions for callbacks that should retain surrounding `this`; use regular methods when an object's receiver is meaningful.
- `bind`, `call`, and `apply` explicitly choose the `this` value of a regular function.
- Default parameters apply when an argument is `undefined`, not when it is `null`.

## Application

Prefer simple functions whose inputs and outputs are explicit. In React function components, `this` is generally irrelevant; know it well enough to diagnose older class components, object methods, and callback APIs.

## Common Mistakes

- Extracting an object method and expecting it to retain its receiver.
- Using an arrow function as an object method when it needs the object as `this`.
- Assuming arrow functions can be used as constructors.

## Common Interview Questions

### Foundation

- How is `this` determined for a regular function?
- How do arrow functions differ from regular functions?

### Intermediate

- Why does `const run = service.run; run()` often fail?
- When would you use `bind`?

### Advanced and Follow-up

- Why are arrow callbacks often useful inside a class method?

### Code Prediction

Predict `this` for `user.print()`, for a detached `const print = user.print; print()`, and for an arrow created inside `user.print`.

## Practical Tasks

- Repair a detached callback that loses its receiver.
- Choose between an arrow callback and a regular method for a stated API.

## Readiness Criteria

You can determine `this` from a call site and select arrow or regular functions for their actual semantics.

## References

- [MDN: Functions](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Functions)
- [MDN: Arrow functions](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Functions/Arrow_functions)