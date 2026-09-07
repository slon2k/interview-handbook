# Destructuring, Spread, Rest, Optional Chaining, and Nullish Coalescing

## Definition

These are the modern syntax features that make everyday data transformation in JavaScript (and React, later in this track) concise and readable: destructuring extracts values from objects/arrays into named variables; spread (`...`) expands an iterable/object into individual elements; rest (`...`, the same syntax, opposite direction) collects multiple elements into one array/object; optional chaining (`?.`) and nullish coalescing (`??`) guard against `null`/`undefined` without verbose manual checks.

```javascript
const { name, age = 18 } = user;              // destructuring, with a default
const [first, ...rest] = items;                 // array destructuring + rest
const merged = { ...defaults, ...overrides };       // spread — merges objects, right-hand side wins on conflicts
const city = user?.address?.city ?? "Unknown";        // optional chaining + nullish coalescing
```

## Alternatives & Trade-offs

Writing the equivalent logic manually (`user.name`, `Object.assign({}, defaults, overrides)`, nested `if` checks before accessing `user.address.city`) works identically but is more verbose and easier to get subtly wrong — a manual merge can forget a property, and a manual null-check chain can miss one level. The modern syntax is more concise and, once familiar, more reliable, at the cost of needing to actually learn the (fairly small) set of rules each operator follows.

## How It Works

### Destructuring — extracting values by name or position, with defaults and renaming

```javascript
const user = { name: "Alice", role: "admin" };
const { name, role, isActive = false } = user; // isActive gets the default, since it's not in `user`
const { name: userName } = user;                  // renamed while destructuring

const coordinates = [10, 20, 30];
const [x, y, z] = coordinates;
const [first, , third] = coordinates;                // skip an element with an empty slot

function greet({ name, role = "guest" }) {              // destructuring directly in a function parameter
  console.log(`${name} (${role})`);
}
greet({ name: "Bob" }); // "Bob (guest)"
```

### Spread — expanding a value into individual elements

```javascript
const arr1 = [1, 2];
const arr2 = [3, 4];
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4] — NOT [arr1, arr2], each array is expanded, not nested

const base = { a: 1, b: 2 };
const extended = { ...base, b: 99, c: 3 }; // { a: 1, b: 99, c: 3 } — later keys win when spreading objects

function sum(a, b, c) { return a + b + c; }
const nums = [1, 2, 3];
sum(...nums); // spreads the array's elements as individual ARGUMENTS: sum(1, 2, 3)
```

### Rest — the same `...` syntax, collecting instead of expanding

```javascript
function logAll(first, ...others) { // `first` gets the first argument, `others` collects the REST as an array
  console.log(first, others);
}
logAll(1, 2, 3, 4); // 1, [2, 3, 4]

const { id, ...otherFields } = { id: 1, name: "Widget", price: 9.99 };
console.log(otherFields); // { name: "Widget", price: 9.99 } — everything EXCEPT id
```

Rest and spread use identical `...` syntax but mean opposite things depending on position: on the left of an assignment (destructuring) it *collects*; used to expand an existing array/object into a new one or into function arguments, it *spreads*.

### Optional chaining — short-circuits to `undefined` instead of throwing

```javascript
const user = { address: null };

user.address.city;      // TypeError: Cannot read properties of null — throws immediately
user.address?.city;         // undefined — short-circuits safely instead of throwing

user.getAge?.();               // calls getAge() ONLY if it exists; otherwise evaluates to undefined, no error
user.orders?.[0];                 // safely accesses the first order, or undefined if orders itself is missing
```

### Nullish coalescing — the correct default-value operator (see also the coercion topic's `0`/`""` trap)

```javascript
const settings = { volume: 0, theme: null };

settings.volume || 50;    // 50 — WRONG if 0 is a valid volume; || treats it as falsy
settings.volume ?? 50;       // 0  — RIGHT; ?? only falls back for null/undefined, not other falsy values
settings.theme ?? "light";       // "light" — theme really is null, so the fallback correctly applies here
```

## Application

Use destructuring to extract exactly the fields a function or component actually needs, with defaults instead of manual fallback logic. Use spread for merging and copying objects/arrays instead of manual property-by-property logic. Use optional chaining for any property access where an intermediate value might legitimately be `null`/`undefined`. Use `??` instead of `||` whenever `0`, `""`, or `false` could be a legitimate value rather than "missing."

## Common Mistakes

- Using `||` for a default value where the left side could legitimately be `0`, `""`, or `false`, instead of the more precise `??`.
- Forgetting that spreading objects applies keys left to right, so an earlier spread's properties can unintentionally overwrite a later, more specific one if the order is backwards.
- Chaining `?.` so deeply that a genuinely broken data shape is silently swallowed into `undefined` everywhere, hiding a bug that should have surfaced.
- Confusing rest and spread as "the same thing" rather than recognizing they're the same syntax used in opposite directions (collecting vs. expanding).

## Common Interview Questions

### Basic
- What's the difference between rest and spread, given they use the same `...` syntax?
- What does optional chaining (`?.`) do when a property doesn't exist?

### Intermediate
- Why is `??` generally preferred over `||` for default values?
- How would you merge two objects, with the second object's properties taking priority on conflicts?

### Advanced
- Walk through why `{...a, ...b}` and `{...b, ...a}` can produce different results if `a` and `b` share a key.
- When might overusing optional chaining hide a bug that should have been surfaced instead?

### Follow-up Questions
- Does optional chaining work for function calls, not just property access?
- Can destructuring provide a default value for a property that's `null`, or only for one that's `undefined`?

### Code Prediction
```javascript
const config = { retries: 0, timeout: null };
const retries = config.retries ?? 3;
const retriesOr = config.retries || 3;
console.log(retries, retriesOr);
```
Predict both values, and explain why they differ given `config.retries` is `0`.

## Practical Tasks

- Rewrite a manual object-merge function using spread, and verify property-override order matches the intended priority.
- Replace a chain of manual `if (obj && obj.a && obj.a.b)` null-guards with optional chaining.
- Fix a default-value expression using `||` that incorrectly treats a legitimate `0` or `""` as missing, by switching to `??`.

## Readiness Criteria

Use destructuring, spread, and rest correctly and distinguish their directions, apply optional chaining and nullish coalescing precisely, and explain why `??` is usually the safer default-value choice over `||`.

## References

- [MDN: Destructuring assignment](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
- [MDN: Spread syntax](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Spread_syntax)
- [MDN: Optional chaining](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Optional_chaining)
