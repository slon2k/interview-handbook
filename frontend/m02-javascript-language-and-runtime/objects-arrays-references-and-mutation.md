# Objects, Arrays, References, and Mutation

## Definition

Objects and arrays are stored and passed by reference — a variable holding an object doesn't hold the object itself, it holds a pointer to it. Assigning, passing to a function, or storing an object in another structure copies the *reference*, not the object's contents, which means two variables can end up pointing at the same underlying data without either one obviously creating a copy.

```javascript
const original = { name: "Alice" };
const copy = original;       // copies the REFERENCE, not the object
copy.name = "Bob";
console.log(original.name);   // "Bob" — original was mutated too, since copy and original are the SAME object
```

## Alternatives & Trade-offs

Mutating objects and arrays in place is fast and avoids allocating new data structures, but makes it easy to accidentally change something another part of the code still holds a reference to — especially dangerous in UI code (React, covered later in this track) where unexpected mutation can cause stale renders or subtle bugs that don't show up immediately. Treating data as immutable (always producing a new object/array instead of mutating) makes changes predictable and traceable, at the cost of creating more short-lived objects.

## How It Works

### Shallow copying — copies one level deep, nested objects still share references

```javascript
const user = { name: "Alice", address: { city: "Berlin" } };

const shallowCopy = { ...user };       // spread creates a NEW top-level object
shallowCopy.name = "Bob";
console.log(user.name);                 // "Alice" — top-level property was NOT shared

shallowCopy.address.city = "Munich";
console.log(user.address.city);           // "Munich" — nested object IS still shared, since spread only copies one level deep!
```

This is one of the most common real bugs in UI code: a spread (`{...obj}`) feels like "a full copy," but only the top level is actually new — any nested object or array inside it is still the exact same reference as before.

### Mutating array methods vs. non-mutating array methods

```javascript
const items = [3, 1, 2];

items.sort();          // MUTATES items in place, and also returns it
items.push(4);            // MUTATES items in place

const sorted = [...items].sort();  // copies FIRST, then sorts the copy — original items unaffected
const withNewItem = [...items, 4];    // never mutates — always produces a new array
```

`sort()`, `push()`, `pop()`, `splice()`, and `reverse()` all mutate the original array and are a common source of "why did this other part of my code suddenly change" bugs when the same array reference is shared or passed around.

### Deep cloning, when a true independent copy is actually needed

```javascript
const user = { name: "Alice", address: { city: "Berlin" } };

const deepCopy = structuredClone(user); // a real, fully independent deep copy, built into modern JS runtimes
deepCopy.address.city = "Munich";
console.log(user.address.city);           // "Berlin" — the original is genuinely untouched this time
```

`structuredClone()` is the modern, correct way to deep-clone plain data — older code sometimes used `JSON.parse(JSON.stringify(obj))` for the same purpose, which works for simple data but silently drops functions, `undefined` values, and `Date` objects (converting them to strings).

### Passing objects to functions — the same reference-sharing rule applies

```javascript
function addAdminFlag(user) {
  user.isAdmin = true; // MUTATES the caller's original object — no copy was ever made
  return user;
}

const currentUser = { name: "Alice" };
addAdminFlag(currentUser);
console.log(currentUser.isAdmin); // true — the caller's own object was changed, even though it looks like a "return value" was used
```

Objects and arrays are passed to functions the same way they're assigned to variables — by reference — so a function that mutates a parameter mutates the caller's actual data, which can be surprising if the caller expected the original argument to be left alone.

## Application

Prefer non-mutating operations (spread, `map`, `filter`, `toSorted`/`toSpliced`) when producing new data from existing data, especially in UI code where mutation can hide state changes from a framework's change detection. Use `structuredClone()` when a genuinely independent deep copy is required, not a shallow spread. Be explicit in function design about whether a function mutates its argument or returns a new value — never leave it ambiguous.

## Common Mistakes

- Assuming `{ ...obj }` or `[...arr]` produces a full deep copy, then being surprised that a nested object inside it is still shared with the original.
- Using a mutating array method (`sort`, `push`, `splice`) on an array that's shared elsewhere, causing an unrelated part of the code to see the change unexpectedly.
- Writing a function that mutates its object parameter without documenting or expecting that behavior, surprising callers who assumed their original data was untouched.
- Using `JSON.parse(JSON.stringify(obj))` for deep cloning without realizing it silently drops functions and `undefined` values and converts `Date` objects to strings.

## Common Interview Questions

### Basic
- What's the difference between how primitives and objects are copied when assigned to a new variable?
- What does "shallow copy" mean, and how does spread (`{...obj}`) relate to it?

### Intermediate
- Why can mutating an array with `sort()` or `push()` cause bugs in code that shares that array reference elsewhere?
- How would you create a genuinely independent deep copy of a nested object?

### Advanced
- Walk through why `{ ...user }` followed by mutating a nested property still affects the original object.
- Why is mutation especially risky in UI code that relies on detecting whether data has changed?

### Follow-up Questions
- Does `structuredClone()` handle functions or `undefined` values in the object being cloned?
- Is passing an object to a function the same, reference-sharing behavior as assigning it to a new variable?

### Code Prediction
```javascript
const state = { count: 0, items: ["a", "b"] };
const next = { ...state, count: state.count + 1 };
next.items.push("c");
console.log(state.items);
```
What does this log, and why, given that `next` was created with a spread from `state`?

## Practical Tasks

- Reproduce the shallow-copy nested-mutation bug, then fix it using `structuredClone()` or a deeper, explicit copy.
- Rewrite a function that mutates its array/object parameter to instead return a new value without touching the original.
- Identify every mutating array method used in a sample codebase and replace each with its non-mutating equivalent.

## Readiness Criteria

Explain reference versus value semantics for objects and primitives precisely, distinguish shallow from deep copying with concrete examples, and consistently avoid unintended mutation in shared data.

## References

- [MDN: Working with objects](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Working_with_objects)
- [MDN: structuredClone()](https://developer.mozilla.org/docs/Web/API/Window/structuredClone)
- [MDN: Array methods that mutate](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array#mutator_methods)
