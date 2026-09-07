# Values, Types, and Equality

## Definition

JavaScript has seven primitive types (`string`, `number`, `boolean`, `undefined`, `null`, `symbol`, `bigint`) and one composite type, `object` (which arrays, functions, and dates all are, under the hood). Primitives are compared and copied by value; objects are compared and copied by reference. `===` (strict equality) compares without converting types; `==` (loose equality) coerces one or both operands first, following rules that are easy to get wrong.

```javascript
typeof "hello"   // "string"
typeof 42         // "number"
typeof true        // "boolean"
typeof undefined     // "undefined"
typeof null           // "object"  — a famous, long-standing JavaScript bug, kept for backward compatibility
typeof Symbol()         // "symbol"
typeof {}                 // "object"
typeof []                   // "object" — arrays are objects; use Array.isArray() to distinguish them
typeof function(){}           // "function" — a special case, technically still an object underneath
```

## Alternatives & Trade-offs

`==` performs implicit coercion before comparing, which can shorten some comparisons (`if (value == null)` catches both `null` and `undefined` in one check) but produces surprising results in most other cases. `===` never coerces, making comparisons predictable at the cost of needing to handle `null`/`undefined` explicitly when both are genuinely possible. Most style guides mandate `===` by default, reserving `== null` as the one deliberate, well-known exception.

## How It Works

### Primitives compare by value; objects compare by reference

```javascript
"abc" === "abc"        // true — same value
5 === 5                  // true — same value

{} === {}                  // false — two DIFFERENT objects, even with identical (empty) content
[1, 2] === [1, 2]             // false — two different arrays, even with identical content

const a = { x: 1 };
const b = a;
a === b                         // true — SAME reference, not just equal content
```

This is the single most common source of "why doesn't this equality check work" confusion in JavaScript: comparing two objects with `===` checks whether they're the *same object in memory*, not whether their contents match.

### `==` coercion rules — surprising results worth knowing by name

```javascript
0 == false        // true  — false coerces to 0
"" == false         // true  — "" coerces to 0, false coerces to 0
null == undefined     // true  — a special-cased rule, these two ARE loosely equal to each other
null == 0                // false — null does NOT coerce to 0 for this comparison, despite the rule above
[] == false                 // true  — [] coerces to "" (via toString), then to 0
[] == ![]                     // true  — a classic "gotcha" example, combining coercion and negation
```

These specific results are memorized more than derived — the practical lesson isn't "learn every coercion rule" but "use `===` and avoid needing to know them at all."

### `Object.is()` — stricter than `===` for two specific edge cases

```javascript
NaN === NaN          // false — NaN is never equal to itself under ===
Object.is(NaN, NaN)    // true

0 === -0                 // true — positive and negative zero are === equal
Object.is(0, -0)            // false
```

`Object.is()` is rarely needed day to day, but explains why `Array.prototype.includes()` (which uses `Object.is`-like semantics) can find `NaN` in an array while `indexOf()` (which uses `===`) cannot.

### The one common, deliberate use of `==`

```javascript
if (value == null) { /* catches BOTH null and undefined in a single check */ }
// equivalent to, but shorter than:
if (value === null || value === undefined) { }
```

## Application

Use `===` by default for all comparisons. Reserve `== null` as the one well-known, deliberate exception for catching both `null` and `undefined` together. Use `typeof` for primitives, but `Array.isArray()` specifically to distinguish arrays from other objects, since `typeof` reports both as `"object"`.

## Common Mistakes

- Using `==` by default and being surprised by coercion results like `"" == false` or `[] == false`.
- Comparing two objects or arrays with `===` expecting a content comparison, when it actually checks reference identity.
- Using `typeof value === "object"` to check for a plain object, forgetting this is also true for arrays and `null`.
- Not knowing that `NaN === NaN` is `false`, and using `===` to check for `NaN` instead of `Number.isNaN()`.

## Common Interview Questions

### Basic
- What's the difference between `==` and `===`?
- What does `typeof null` return, and why is that considered a bug?

### Intermediate
- Why does comparing two arrays with identical content using `===` return `false`?
- What's the correct way to check if a value is `NaN`?

### Advanced
- Walk through why `[] == false` evaluates to `true`, step by step through the coercion rules.
- When, if ever, is `==` the more correct choice over `===`?

### Follow-up Questions
- Is `null == undefined` true or false, and is that consistent with `null == 0`?
- Does `Object.is()` behave identically to `===` for all values?

### Code Prediction
```javascript
console.log(0 == "0");
console.log(0 == []);
console.log("0" == []);
```
Predict the output of each line, and explain the coercion path that produces it.

## Practical Tasks

- Rewrite a set of `==` comparisons as `===`, adding explicit `null`/`undefined` checks where the loose equality was relying on coercion.
- Write a function that correctly checks for `NaN` without using `===`.
- Explain, for a code review, why two visually identical objects fail a `===` comparison.

## Readiness Criteria

Distinguish `==` from `===` precisely, explain reference versus value comparison for objects versus primitives, and predict common coercion results without needing to guess.

## References

- [MDN: Equality comparisons and sameness](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Equality_comparisons_and_sameness)
- [MDN: Data types and structures](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Data_structures)
