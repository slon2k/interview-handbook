# Modules and Error Handling

## Definition

ES Modules (`import`/`export`) let JavaScript files share code with explicit, statically-analyzable dependencies — the standard module system in modern JavaScript and the one bundlers/TypeScript (later in this track) are built around. Error handling covers both synchronous errors (`try`/`catch`) and the distinct mechanisms needed for asynchronous failures (a rejected promise, an error inside an `async` function), which don't work the same way.

```javascript
// math.js
export function add(a, b) { return a + b; }
export default function multiply(a, b) { return a * b; }

// app.js
import multiply, { add } from "./math.js";
```

## Alternatives & Trade-offs

CommonJS (`require`/`module.exports`), Node.js's older module system, resolves dependencies dynamically at runtime, which is flexible but harder for tools to statically analyze (for tree shaking, or for TypeScript to fully understand types ahead of time). ES Modules are statically analyzable — imports/exports must appear at the top level, not inside conditionals — which is exactly what lets bundlers eliminate unused code (Module 6's tree-shaking content) and lets tooling reason about a module's shape without executing it.

## How It Works

### Named exports vs. default export — and why both exist

```javascript
// utils.js
export const PI = 3.14159;              // named export — a module can have MANY of these
export function square(x) { return x * x; }
export default class Calculator { }        // default export — a module can have only ONE of these

// consuming file
import Calculator, { PI, square } from "./utils.js"; // default has no braces; named exports use braces
import * as Utils from "./utils.js";                   // import everything as one namespace object
```

Named exports are explicit about exactly what's being imported (and get renamed easily: `import { square as sq }`), while a default export is meant for "the one main thing this module provides" — mixing many default exports across a codebase inconsistently is a common source of confusion about what a given import actually refers to.

### Why static imports enable tree shaking

```javascript
// utils.js exports 20 functions; app.js only uses one of them
import { formatDate } from "./utils.js";
```

Because ES Module imports/exports are statically declared (never computed or conditional), a bundler can determine at build time that only `formatDate` is actually used and exclude the other 19 functions from the final bundle entirely — something far harder to do reliably with CommonJS's dynamic `require()` calls.

### `try`/`catch`/`finally` for synchronous errors

```javascript
function parseConfig(json) {
  try {
    return JSON.parse(json);
  } catch (error) {
    console.error("Invalid config JSON:", error.message);
    return null;
  } finally {
    console.log("Parse attempt finished"); // ALWAYS runs, whether an error occurred or not
  }
}
```

### `try`/`catch` does NOT catch errors from unawaited promises

```javascript
function loadData() {
  try {
    fetch("/api/data").then(res => res.json()); // if this promise REJECTS, the catch below never sees it
  } catch (error) {
    console.log("caught:", error); // never runs for a rejected promise here — try/catch only catches SYNCHRONOUS throws
  }
}
```

```javascript
// Correct: await the promise inside the try block so its rejection becomes a catchable throw
async function loadData() {
  try {
    const res = await fetch("/api/data");
    return await res.json();
  } catch (error) {
    console.log("caught:", error); // now this DOES catch a fetch failure or a rejected promise
  }
}
```

This is one of the most common real async bugs: `try`/`catch` only catches errors thrown *synchronously* during the try block's execution — a promise that rejects *later*, without being awaited inside that same try block, sails right past it.

### Custom error types — carrying more information than a plain string

```javascript
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}

function validateAge(age) {
  if (age < 0) throw new ValidationError("Age cannot be negative", "age");
}

try {
  validateAge(-5);
} catch (error) {
  if (error instanceof ValidationError) {
    console.log(`Validation failed on field: ${error.field}`);
  } else {
    throw error; // re-throw anything that ISN'T the specific error type this catch block knows how to handle
  }
}
```

## Application

Use named exports for most module contents, reserving a default export for a module's single primary thing (a component, a main class). Always `await` a promise inside the same `try` block that's meant to catch its failure — a `.then()` chain or an unawaited async call inside a `try` will not be caught by that `try`'s `catch`. Use custom error classes when different failure types need to be handled differently, not just logged identically.

## Common Mistakes

- Wrapping a `fetch(...).then(...)` chain in `try`/`catch` without `await`ing it, expecting the `catch` to handle a later rejection that it can never actually see.
- Mixing default and named exports inconsistently across a codebase, making it unclear what each import actually refers to without checking the source file.
- Catching every error identically regardless of type, instead of checking `error instanceof SpecificErrorType` to handle different failures differently.
- Forgetting to re-throw an error in a `catch` block that only knows how to handle one specific error type, silently swallowing an unrelated failure instead.

## Common Interview Questions

### Basic
- What's the difference between a named export and a default export?
- What does `finally` guarantee, regardless of whether an error occurred?

### Intermediate
- Why doesn't `try`/`catch` catch a promise that rejects inside an unawaited `.then()` chain?
- How does static module analysis (ES Modules) enable tree shaking in a way CommonJS makes harder?

### Advanced
- Walk through why wrapping an unawaited `fetch(...).then(...)` in `try`/`catch` fails to catch a network error.
- How would you design a custom error hierarchy to distinguish validation errors from network errors in a catch block?

### Follow-up Questions
- Can a module have both a default export and named exports at the same time?
- Does `finally` run if the `try` block returns a value?

### Code Prediction
```javascript
async function risky() {
  try {
    fetchDataAsync().then(data => { throw new Error("processing failed"); });
  } catch (error) {
    console.log("Caught:", error.message);
  }
  console.log("Function completed");
}
```
Assuming `fetchDataAsync()` resolves successfully and the `.then()` callback then throws, does `"Caught: processing failed"` ever get logged? What does get logged, and in what order?

## Practical Tasks

- Fix a `try`/`catch` block that fails to catch an async error by adding the missing `await`.
- Design a small custom error class hierarchy distinguishing at least two different failure types, and handle each differently in a `catch` block.
- Convert a CommonJS-style module (`require`/`module.exports`) to ES Module syntax (`import`/`export`).

## Readiness Criteria

Use named and default exports appropriately, correctly identify why `try`/`catch` misses unawaited async errors and fix it, and design custom error types for differentiated error handling.

## References

- [MDN: JavaScript modules](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules)
- [MDN: try...catch](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/try...catch)
- [MDN: Error](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Error)
