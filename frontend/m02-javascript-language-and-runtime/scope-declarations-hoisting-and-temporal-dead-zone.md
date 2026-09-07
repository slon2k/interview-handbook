# Scope, Declarations, Hoisting, and the Temporal Dead Zone

## Definition

Scope determines where a binding is visible. `var` is function-scoped and hoisted with an initial value of `undefined`; `let`/`const` are block-scoped and hoisted too, but remain unusable — in the **temporal dead zone (TDZ)** — from the top of their block until the actual declaration line executes. This difference is why `var`'s loop-variable-capture bug (a classic closure gotcha) doesn't happen with `let`.

```javascript
console.log(a); // undefined — var IS hoisted, with an initial value of undefined
var a = 1;

console.log(b); // ReferenceError: Cannot access 'b' before initialization — TDZ
let b = 2;
```

## Alternatives & Trade-offs

`var`'s function-scoping and hoisting-to-`undefined` behavior is more permissive — code technically runs even when a variable is referenced before its declaration — but that permissiveness is exactly what causes real, hard-to-spot bugs (the classic loop-closure trap below). `let`/`const`'s block-scoping and TDZ enforcement are stricter, throwing an error immediately when a variable is used before it's ready, which surfaces the same class of bug loudly and immediately instead of silently producing `undefined`.

## How It Works

### `var` is function-scoped; `let`/`const` are block-scoped

```javascript
function example() {
  if (true) {
    var functionScoped = "visible outside this block";
    let blockScoped = "only visible inside this block";
  }
  console.log(functionScoped); // "visible outside this block" — var ignores the if-block entirely
  console.log(blockScoped);       // ReferenceError: blockScoped is not defined
}
```

`var` only respects function boundaries, not `if`/`for`/block boundaries — a `var` declared inside an `if` block is visible throughout the entire enclosing function, which routinely surprises people coming from block-scoped languages.

### The classic `var`-in-a-loop closure bug, and why `let` fixes it

```javascript
const callbacks = [];
for (var i = 0; i < 3; i++) {
  callbacks.push(() => console.log(i));
}
callbacks.forEach(cb => cb()); // logs 3, 3, 3 — every closure captured the SAME shared `var i`

const callbacksFixed = [];
for (let j = 0; j < 3; j++) {
  callbacksFixed.push(() => console.log(j));
}
callbacksFixed.forEach(cb => cb()); // logs 0, 1, 2 — let creates a NEW binding for EACH loop iteration
```

This is one of the most frequently tested JavaScript interview snippets. With `var`, there's only one shared `i` across the entire loop, so every callback closes over that same final value. With `let`, the language creates a fresh binding of `j` for each iteration, so each callback captures its own independent value.

### Hoisting — declarations move up, but only partially

```javascript
console.log(typeof myFunc); // "function" — function DECLARATIONS are hoisted completely, body included
function myFunc() {}

console.log(typeof myVar);  // "undefined" — var is hoisted, but only the declaration, not the assignment
var myVar = "value";

console.log(typeof myConst); // ReferenceError — let/const are hoisted too, but land in the TDZ until reached
const myConst = "value";
```

Function *declarations* (`function myFunc() {}`) are hoisted with their entire body attached, so they can be called before their line in the source. Function *expressions* (`const myFunc = function() {}`) follow variable-hoisting rules instead, since they're really just a variable assignment.

### The temporal dead zone, precisely

```javascript
{
  // TDZ for `x` begins here — the block has started, but x's declaration line hasn't run yet
  console.log(x); // ReferenceError: Cannot access 'x' before initialization
  let x = 5;       // TDZ for x ENDS here
  console.log(x);  // 5 — fine now
}
```

The TDZ isn't "the variable doesn't exist yet" — it technically does exist (that's why it's a distinct `ReferenceError` message, not "undefined variable") — it's "the variable exists but accessing it is forbidden until its declaration is actually reached."

## Application

Use `const` by default, `let` when reassignment is genuinely needed, and avoid `var` entirely in new code — its function-scoping and hoisting-to-`undefined` behavior are exactly what cause the classic loop-closure bug and other hard-to-spot mistakes that block-scoping and the TDZ prevent by failing loudly instead.

## Common Mistakes

- Using `var` inside a loop that creates closures (event handlers, callbacks, `setTimeout`), producing the classic "every callback sees the same final value" bug.
- Assuming a `var` declared inside an `if` or `for` block is scoped to that block, when it's actually visible throughout the whole enclosing function.
- Confusing the TDZ's `ReferenceError` with "the variable doesn't exist," when it actually means the variable exists but hasn't been initialized yet.
- Assuming function expressions (`const fn = function() {}`) are hoisted the same way function declarations are.

## Common Interview Questions

### Basic
- What's the difference between function-scoping and block-scoping?
- What is the temporal dead zone?

### Intermediate
- Why does using `var` in a `for` loop with a closure produce unexpected results, and how does `let` fix it?
- Are function declarations hoisted the same way as function expressions?

### Advanced
- Walk through, step by step, why `for (var i = 0; ...)` with a closure logs the same final value for every callback.
- What's the practical difference between "the variable doesn't exist" and "the variable is in the TDZ"?

### Follow-up Questions
- Does `const` prevent reassignment of an object's properties, or only reassignment of the variable itself?
- Is `let` hoisted at all, or does it only start existing at its declaration line?

### Code Prediction
```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 0);
}
```
Predict the full sequence of logged values from both loops, in order.

## Practical Tasks

- Reproduce the `var`-in-a-loop closure bug with `setTimeout`, then fix it by switching to `let`.
- Identify a `var` declaration inside a conditional block that's actually being relied upon outside that block, and refactor it to explicit block-scoped `let`/`const`.
- Explain, for a code review, why a `ReferenceError: Cannot access before initialization` is different from "variable is not defined."

## Readiness Criteria

Explain function-scoping versus block-scoping precisely, predict the classic `var`-loop-closure bug and its `let`-based fix, and explain hoisting and the TDZ without conflating "hoisted" with "usable immediately."

## References

- [MDN: let](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/let)
- [MDN: Closures — loops and closures](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Closures#creating_closures_in_loops_a_common_mistake)
- [MDN: Hoisting](https://developer.mozilla.org/docs/Glossary/Hoisting)
