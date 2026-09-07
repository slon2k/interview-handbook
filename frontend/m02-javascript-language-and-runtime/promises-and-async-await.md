# Promises and Async/Await

## Definition

A Promise represents the eventual result (or failure) of an asynchronous operation, in one of three states: pending, fulfilled, or rejected. `async`/`await` is syntax sugar over promises — an `async` function always returns a promise, and `await` pauses that function (without blocking the rest of the program) until the awaited promise settles, letting asynchronous code read like sequential, synchronous code.

```javascript
function fetchUser(id) {
  return fetch(`/api/users/${id}`).then(res => res.json()); // returns a Promise
}

async function loadUser(id) {
  const user = await fetchUser(id); // pauses HERE until the promise resolves, without blocking the browser
  console.log(user);
}
```

## Alternatives & Trade-offs

Raw `.then()`/`.catch()` chains work identically to `async`/`await` underneath, but nested or branching chains become hard to read, and error handling requires a `.catch()` at the right point in the chain. `async`/`await` reads top-to-bottom like synchronous code and lets a single `try`/`catch` wrap multiple awaited steps, at the cost of needing to understand that "looks synchronous" doesn't mean "blocks the thread" — it's still fundamentally the same non-blocking promise machinery underneath.

## How It Works

### The three states, and why a promise can only settle once

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => resolve("done"), 1000); // resolves ONCE, after 1 second
});

promise.then(result => console.log(result)); // "done", after ~1 second
```

Once a promise resolves or rejects, its state is permanent — calling `resolve` or `reject` again afterward has no effect, and every `.then()`/`await` attached to it (even ones added later) sees the same final result.

### Sequential vs. concurrent `await` — a very common real mistake

```javascript
// Sequential: each await BLOCKS the next line from starting until it resolves — slower than necessary
async function loadBoth() {
  const user = await fetchUser(1);      // waits fully before starting the next line
  const posts = await fetchPosts(1);       // only starts AFTER user has already resolved
  return { user, posts };
}

// Concurrent: both requests start immediately, THEN we wait for both together — faster
async function loadBothConcurrently() {
  const userPromise = fetchUser(1);     // starts immediately, not yet awaited
  const postsPromise = fetchPosts(1);      // ALSO starts immediately, in parallel with the line above
  const [user, posts] = await Promise.all([userPromise, postsPromise]);
  return { user, posts };
}
```

If `fetchUser` and `fetchPosts` don't depend on each other's results, awaiting them one at a time makes the total wait time the *sum* of both requests, when it could be the *longer* of the two — a real, common performance mistake that's easy to introduce by simply writing two `await` lines in sequence without thinking about whether they need to be.

### `Promise.all` vs. `Promise.allSettled` — how failure is handled differently

```javascript
Promise.all([fetchUser(1), fetchPosts(1)])
  .then(([user, posts]) => { })
  .catch(error => console.log("at least one failed:", error)); // if EITHER rejects, this whole thing rejects

Promise.allSettled([fetchUser(1), fetchPosts(1)])
  .then(results => {
    // results is an array of { status: "fulfilled", value } or { status: "rejected", reason } —
    // NEVER rejects itself, even if every individual promise failed
    results.forEach(r => console.log(r.status));
  });
```

`Promise.all` fails fast — the first rejection short-circuits the whole group. `Promise.allSettled` waits for every promise to finish regardless of outcome, useful when partial success (some requests succeeded, some didn't) is a meaningful result rather than an all-or-nothing failure.

### Error handling — `try`/`catch` around `await`, or `.catch()` on the promise chain

```javascript
async function loadUser(id) {
  try {
    const user = await fetchUser(id);
    return user;
  } catch (error) {
    console.error("Failed to load user:", error.message);
    return null;
  }
}
```

A rejected promise that's `await`ed inside a `try` block behaves exactly like a synchronous `throw` for that `try`/`catch` — this is the mechanism that makes `async`/`await`'s error handling feel identical to ordinary synchronous error handling, in contrast to the modules-and-error-handling topic's unawaited-promise trap.

### `async` functions always return a promise, even for a plain return value

```javascript
async function getValue() { return 42; }
getValue();          // returns a Promise that resolves to 42, NOT the number 42 directly
getValue().then(v => console.log(v)); // 42
```

## Application

Use `Promise.all` (or `Promise.allSettled` when partial success matters) to run independent asynchronous operations concurrently instead of `await`ing them one after another. Wrap `await`ed calls in `try`/`catch` for error handling. Remember that an `async` function's return value is always wrapped in a promise, even when the function body itself never explicitly creates one.

## Common Mistakes

- Awaiting independent asynchronous calls one after another instead of starting them concurrently with `Promise.all`, needlessly adding their durations together.
- Using `Promise.all` when partial success is actually acceptable, causing the entire operation to fail because of one rejected promise that shouldn't have blocked the others.
- Forgetting that an `async` function's return value is a promise, and trying to use it directly without `await` or `.then()`.
- Not wrapping an `await`ed call in `try`/`catch`, leaving an unhandled promise rejection when the awaited operation fails.

## Common Interview Questions

### Basic
- What are the three states a promise can be in?
- What does `await` actually do inside an `async` function?

### Intermediate
- Why does awaiting two independent requests sequentially take longer than necessary, and how do you fix it?
- What's the difference between `Promise.all` and `Promise.allSettled`?

### Advanced
- Walk through why an `async` function that just does `return 42` doesn't return the number `42` directly to its caller.
- How would you handle a scenario where three independent requests should all be attempted, but the overall operation should succeed even if one of them fails?

### Follow-up Questions
- Can a promise be resolved and then later rejected, or does its first settlement stick permanently?
- Does `await` block the browser's main thread while waiting?

### Code Prediction
```javascript
async function slow() { await new Promise(r => setTimeout(r, 100)); return "slow"; }
async function fast() { await new Promise(r => setTimeout(r, 10)); return "fast"; }

async function sequential() {
  const a = await slow();
  const b = await fast();
  return [a, b];
}
```
Roughly how long does `sequential()` take to resolve, and how would rewriting it to start both promises before awaiting either change that timing?

## Practical Tasks

- Rewrite a function that sequentially awaits two independent API calls to run them concurrently with `Promise.all`, and measure the time difference.
- Implement error handling for three concurrent requests where partial failure is acceptable, using `Promise.allSettled`.
- Add `try`/`catch` to an `async` function that currently has no error handling around its `await` calls.

## Readiness Criteria

Explain promise states and settlement precisely, correctly reshape sequential `await` code into concurrent execution when appropriate, and choose between `Promise.all` and `Promise.allSettled` based on whether partial failure is acceptable.

## References

- [MDN: Using promises](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Using_promises)
- [MDN: async function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/async_function)
- [MDN: Promise.all()](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)
