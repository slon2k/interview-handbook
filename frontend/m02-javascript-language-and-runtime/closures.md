# Closures

## Definition

A closure is a function bundled together with access to the variables from its enclosing scope, retained even after that outer scope has finished executing. A closure captures *bindings*, not frozen snapshots of their values — if the captured variable changes later, the closure sees the new value, which is exactly what causes both its most useful patterns and its most common bugs.

```javascript
function createCounter() {
  let count = 0;               // this variable is captured by the returned function
  return function increment() {
    count++;                     // reads and updates the OUTER count, even after createCounter has returned
    return count;
  };
}

const counterA = createCounter();
const counterB = createCounter();
console.log(counterA()); // 1
console.log(counterA()); // 2 — counterA's own, independent `count`
console.log(counterB()); // 1 — a COMPLETELY separate count, from a different call to createCounter
```

## Alternatives & Trade-offs

Closures give a lightweight way to create private state and parameterized behavior without a class — `createCounter()`'s `count` variable is genuinely inaccessible from outside except through the returned function. A class with a private field accomplishes something similar with more ceremony but clearer intent for larger, multi-method objects; closures are usually the simpler choice for small, single-purpose stateful functions or factories.

## How It Works

### Each call to the outer function creates a distinct, independent environment

```javascript
function createCounter() {
  let count = 0;
  return () => ++count;
}

const a = createCounter();
const b = createCounter();
a(); a(); a(); // count for `a`'s closure is now 3
b();             // count for `b`'s closure is only 1 — entirely separate memory, despite identical code
```

This is precisely the exercise most interviews ask directly: two independent calls to a factory function produce two independent closures, each with its own private copy of the captured variable — not a shared one.

### Closures capture bindings, not values — the classic pitfall

```javascript
function createLoggers() {
  const loggers = [];
  for (var i = 0; i < 3; i++) {
    loggers.push(() => console.log(i)); // captures the VARIABLE i, not its value at push-time
  }
  return loggers;
}

createLoggers().forEach(log => log()); // 3, 3, 3 — all three closures share the SAME final `i`
```

This is the exact same underlying mechanism as the `var`-in-a-loop bug from the scope/hoisting topic — closures don't "freeze" a value at the moment they're created; they keep a live reference to the actual variable, so if that variable is later shared and mutated (as `var i` is across loop iterations), every closure over it sees the final state.

### Using closures for genuinely private state

```javascript
function createBankAccount(initialBalance) {
  let balance = initialBalance; // truly private — no external code can reach this variable directly
  return {
    deposit(amount) { balance += amount; },
    withdraw(amount) { if (amount <= balance) balance -= amount; },
    getBalance() { return balance; }
  };
}

const account = createBankAccount(100);
account.deposit(50);
console.log(account.getBalance()); // 150
console.log(account.balance);         // undefined — there is NO way to access balance except through the returned methods
```

### Closures and memory — why long-lived closures can matter

```javascript
function attachHandler(largeData) {
  const summary = largeData.length; // only this small value is actually needed later

  document.addEventListener("click", () => console.log(summary));
  // if the ENTIRE largeData were referenced inside the closure instead of just `summary`,
  // largeData could be kept alive in memory for as long as this event listener exists,
  // even though only a tiny piece of it was ever actually needed
}
```

A closure keeps everything in its captured scope reachable for as long as the closure itself is reachable — an event listener, timer, or cache holding a closure over a much larger object than it actually needs can prevent that larger object from ever being garbage collected.

## Application

Use closures to create private, encapsulated state for a factory function or module, and to parameterize a callback with data from its creation context. Be deliberate about *which* variables a long-lived closure (an event listener, a timer callback) actually captures, since it keeps everything in its enclosing scope reachable for as long as it exists.

## Common Mistakes

- Assuming a closure captures a value at the moment it's created, rather than a live reference to the variable itself.
- Reproducing the `var`-in-a-loop closure bug, where every closure ends up sharing and reading the same final loop variable.
- Letting a long-lived closure (an event listener, a cache) unintentionally keep a much larger object reachable in memory than it actually needs.
- Not realizing that two separate calls to the same factory function produce two completely independent closures, not two references to shared state.

## Common Interview Questions

### Basic
- What is a closure?
- Why can an inner function still access an outer function's variables after the outer function has returned?

### Intermediate
- How can a closure be used to create private state that's otherwise inaccessible?
- What causes the classic loop-variable closure bug, and how does switching to `let` fix it?

### Advanced
- How can a long-lived closure prevent an object from being garbage collected, even if the closure itself only needs a small piece of that object?
- How would you implement a rate limiter or debounce function using a closure to track state between calls?

### Follow-up Questions
- Do two separate calls to the same factory function share the same closure state, or get independent copies?
- Is it possible for a closure to capture a value that no longer exists?

### Code Prediction
```javascript
function makeMultiplier(factor) {
  return (n) => n * factor;
}
const double = makeMultiplier(2);
const triple = makeMultiplier(3);
console.log(double(5), triple(5));
```
Predict the output, and explain why `double` and `triple` don't interfere with each other despite being created by the same function.

## Practical Tasks

- Implement a `createCounter` factory producing independent counters, and verify two instances don't share state.
- Reproduce the loop-variable closure bug with `var`, then fix it with `let`.
- Implement a small module with genuinely private state (like the bank account example) exposing only specific methods.

## Readiness Criteria

Explain lexical capture precisely (bindings, not snapshots), predict closure-based factory behavior for independent instances, and identify closure-related memory-retention risks in long-lived callbacks.

## References

- [MDN: Closures](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Closures)
- [MDN: Closures — creating closures in loops](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Closures#creating_closures_in_loops_a_common_mistake)
