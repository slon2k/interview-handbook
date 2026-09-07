# Coercion, Truthiness, `null`, and `undefined`

## Definition

JavaScript converts values between types implicitly in many contexts — an `if` condition, a template literal, arithmetic with mixed types — following **truthiness** rules for boolean contexts. `undefined` means a variable was declared but never assigned (or a property/argument doesn't exist); `null` means a deliberate, explicit "no value," assigned intentionally by code.

```javascript
if ("hello") { }    // truthy — runs
if (0) { }             // falsy — does not run
if ("") { }              // falsy — does not run
if ([]) { }                // truthy — an empty array is still an object, and all objects are truthy
if ({}) { }                   // truthy — same reasoning
```

## Alternatives & Trade-offs

Relying on truthiness (`if (value)`) is concise but treats `0`, `""`, `NaN`, `null`, `undefined`, and `false` all identically as "falsy" — which is often not what's actually intended (a form field legitimately containing the number `0` is falsy, but not "empty" in the way an empty string is). Explicit checks (`value === undefined`, `value.length === 0`) are more verbose but say exactly what's being tested, avoiding the class of bugs where a legitimate value like `0` is treated as absent.

## How It Works

### The eight falsy values — everything else is truthy

```javascript
Boolean(false)      // false
Boolean(0)             // false
Boolean(-0)              // false
Boolean(0n)                // false (BigInt zero)
Boolean("")                   // false
Boolean(null)                    // false
Boolean(undefined)                  // false
Boolean(NaN)                           // false

Boolean([])       // true  — an empty array is still an object
Boolean({})          // true  — an empty object is still an object
Boolean("0")            // true  — a NON-empty string, even though it "looks like" zero
```

The `"0"` case is a classic trap: the *string* `"0"` is truthy, even though the *number* `0` is falsy — a common source of bugs when a value coming from an HTML input (always a string) is checked with a plain `if`.

### The `0` truthiness trap in practice

```javascript
function displayCount(count) {
  if (count) {
    return `Count: ${count}`;
  }
  return "No count available";
}

displayCount(0); // "No count available" — probably WRONG; 0 is a legitimate count, not a missing one
```

```javascript
// Fixed: explicitly check for the actual "missing" condition, not just falsiness
function displayCount(count) {
  if (count === undefined || count === null) {
    return "No count available";
  }
  return `Count: ${count}`;
}
```

### `null` vs. `undefined` — who's responsible for each

```javascript
let x;                      // undefined — JavaScript itself assigns this default
console.log(x);                 // undefined

function greet(name) {
  console.log(name);              // undefined, if greet() is called with no argument
}
greet();

const user = { name: "Alice" };
console.log(user.age);                // undefined — accessing a property that doesn't exist

let selectedUser = null;                 // null — a developer EXPLICITLY chose "no value" here
```

The practical convention: `undefined` is what JavaScript gives you automatically when something wasn't provided; `null` is what a developer assigns deliberately to mean "intentionally empty," distinguishing "nothing was ever set" from "this was explicitly cleared."

### Nullish coalescing — distinguishing "falsy" from "actually missing"

```javascript
const count = 0;
console.log(count || "default");    // "default" — WRONG if 0 is a valid value; || treats 0 as falsy
console.log(count ?? "default");       // 0 — RIGHT; ?? only falls back for null or undefined specifically
```

`??` (covered further in the modern-syntax topic) exists specifically to solve the `0`/`""`-truthiness trap: it only substitutes its right-hand value when the left is genuinely `null` or `undefined`, not for any other falsy value.

### Implicit coercion in string and numeric contexts

```javascript
"5" + 3        // "53" — + with a string operand triggers STRING concatenation
"5" - 3           // 2   — - has no string-concatenation meaning, so it triggers NUMERIC coercion instead
"5" * "2"            // 10  — both operands coerce to numbers for *
```

`+` is uniquely ambiguous between string concatenation and numeric addition, and JavaScript resolves that ambiguity by checking if *either* operand is a string — every other arithmetic operator only has a numeric meaning, so it always coerces toward numbers.

## Application

Use explicit checks (`=== undefined`, `.length === 0`) instead of plain truthiness whenever `0`, `""`, or `NaN` could be a legitimate, meaningful value rather than "absence." Use `??` instead of `||` for default values specifically to avoid the `0`/`""`-truthiness trap. Reserve `null` for values a developer explicitly clears or initializes as empty; let `undefined` represent "was never set" without assigning it manually.

## Common Mistakes

- Using `if (value)` to check for "missing," when `0`, `""`, or `NaN` are legitimate values that get incorrectly treated as absent.
- Using `||` for a default value where the left-hand side could legitimately be `0` or `""`, instead of `??`.
- Confusing `null` and `undefined` in code style, assigning `null` in places JavaScript would have already produced `undefined` naturally.
- Assuming `"0"` (the string) is falsy the same way `0` (the number) is.

## Common Interview Questions

### Basic
- What are the eight falsy values in JavaScript?
- What's the difference between `null` and `undefined`?

### Intermediate
- Why does `if (count)` incorrectly treat a `count` of `0` as "no value"?
- What's the difference between `||` and `??` for providing a default value?

### Advanced
- Walk through why `"5" + 3` and `"5" - 3` produce different types of results.
- How would you design a function's default-value logic to correctly handle `0` as a legitimate input?

### Follow-up Questions
- Is the string `"0"` truthy or falsy?
- Does `??` treat `0` the same way `||` does?

### Code Prediction
```javascript
function getDisplayValue(value) {
  return value || "N/A";
}
console.log(getDisplayValue(0));
console.log(getDisplayValue(""));
console.log(getDisplayValue(null));
```
Predict each line's output, and identify which of these results is probably not what the function's author actually intended.

## Practical Tasks

- Fix a default-value function relying on `||` so that a legitimate `0` or `""` input isn't incorrectly replaced with a fallback.
- Audit a sample codebase for `if (value)` checks that should be more specific (`=== undefined`, `.length === 0`) given the actual data involved.
- Explain, for a code review, why `"5" + 3` and `"5" - 3` behave differently.

## Readiness Criteria

Identify the eight falsy values from memory, distinguish `null` from `undefined` by convention and origin, and choose `??` over `||` correctly when `0` or `""` are legitimate values.

## References

- [MDN: Truthy](https://developer.mozilla.org/docs/Glossary/Truthy)
- [MDN: Nullish coalescing operator](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing)
- [MDN: Type coercion](https://developer.mozilla.org/docs/Glossary/Type_coercion)
