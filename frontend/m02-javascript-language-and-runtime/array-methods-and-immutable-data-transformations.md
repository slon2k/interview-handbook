# Array Methods and Immutable Data Transformations

## Definition

Modern JavaScript favors transforming arrays by producing new arrays (`map`, `filter`, `reduce`) rather than mutating the original in place (`push`, `splice`, `sort`) — the same immutability principle from the objects/mutation topic, applied specifically to the small set of array methods used constantly in real UI code.

```javascript
const prices = [10, 20, 30];

const doubled = prices.map(p => p * 2);         // [20, 40, 60] — a NEW array; prices is untouched
const expensive = prices.filter(p => p > 15);        // [20, 30]     — a NEW array
const total = prices.reduce((sum, p) => sum + p, 0);     // 60           — a single accumulated value
```

## Alternatives & Trade-offs

Mutating methods (`push`, `splice`, `sort`, `reverse`) are marginally more efficient (no new array allocated) and were the only option before non-mutating alternatives existed, but they risk exactly the shared-reference bugs covered in the mutation topic — especially dangerous in UI frameworks that decide whether to re-render based on whether an array reference *changed*, not whether its contents did. Non-mutating methods always return a new array, making change detection reliable at the cost of a small, usually negligible allocation cost.

## How It Works

### `map`, `filter`, and `reduce` — the three workhorses

```javascript
const users = [
  { id: 1, name: "Alice", active: true },
  { id: 2, name: "Bob", active: false },
  { id: 3, name: "Carol", active: true }
];

const names = users.map(u => u.name);                        // ["Alice", "Bob", "Carol"]
const activeUsers = users.filter(u => u.active);                 // [{...Alice}, {...Carol}]
const activeCount = users.reduce((count, u) => u.active ? count + 1 : count, 0); // 2
```

`reduce` is the most general of the three — `map` and `filter` can each be expressed as a special case of `reduce`, but are far more readable for their specific, common purposes.

### Updating one item in an array immutably — the pattern used constantly in UI state

```javascript
const todos = [
  { id: 1, text: "Buy milk", done: false },
  { id: 2, text: "Walk dog", done: false }
];

// WRONG: mutates the original array's object in place
todos[0].done = true;

// RIGHT: produces a NEW array, with a NEW object for the changed item, everything else untouched
const updated = todos.map(todo =>
  todo.id === 1 ? { ...todo, done: true } : todo
);
```

This exact pattern — `map` plus a conditional spread — is how React (and most modern UI state management) expects list updates to be expressed, specifically because it produces a genuinely new array and a genuinely new object for the changed item, which is what lets a framework detect the change by reference comparison alone.

### Removing an item immutably

```javascript
const remaining = todos.filter(todo => todo.id !== 1); // a new array without the removed item — no splice needed
```

### Adding an item immutably

```javascript
const withNewTodo = [...todos, { id: 3, text: "Read book", done: false }]; // spread + new item, no push needed
```

### The 2023+ non-mutating array methods — direct replacements for the old mutating ones

```javascript
const original = [3, 1, 2];

const sorted = original.toSorted();     // NEW sorted array; original is untouched (unlike .sort())
const reversed = original.toReversed();    // NEW reversed array; original is untouched (unlike .reverse())
const spliced = original.toSpliced(1, 1);     // NEW array with an item removed; original is untouched (unlike .splice())
console.log(original); // [3, 1, 2] — completely unaffected by any of the above
```

These newer methods exist specifically to provide a non-mutating equivalent for the handful of array operations (`sort`, `reverse`, `splice`) that historically had no non-mutating built-in alternative at all.

### `find` and `some`/`every` — searching without transforming

```javascript
const firstActive = users.find(u => u.active);   // the FIRST matching object itself, or undefined if none match
const hasInactive = users.some(u => !u.active);      // true/false — does AT LEAST ONE match?
const allActive = users.every(u => u.active);           // true/false — do ALL match?
```

## Application

Default to `map`/`filter`/`reduce` (and `toSorted`/`toReversed`/`toSpliced` where sorting or splicing is needed) for any array transformation in UI-adjacent code, so that a genuinely new array/object reference results whenever data actually changes. Reserve mutating methods for cases where the array is genuinely local and not shared or tracked by a framework's change detection.

## Common Mistakes

- Mutating an array or one of its objects in place (`todos[0].done = true`) inside code a UI framework is watching for reference changes, causing the UI to silently fail to re-render.
- Using `.sort()` or `.reverse()` on an array that's also referenced elsewhere, unexpectedly mutating the shared original.
- Reaching for a manual loop with `push()` to build a new array when `map`/`filter` express the same intent more directly and without mutation.
- Confusing `map` (always returns an array of the same length) with `filter` (returns a possibly-shorter array) when the actual goal is to remove items.

## Common Interview Questions

### Basic
- What's the difference between `map` and `filter`?
- Which common array methods mutate the original array, and which don't?

### Intermediate
- How would you update one item in an array of objects without mutating the original array or object?
- What do `toSorted()` and `toSpliced()` provide that `sort()` and `splice()` don't?

### Advanced
- Why does using a mutating array method matter more in UI code than in a simple script?
- How would `reduce` be used to implement the equivalent of `map` or `filter` from scratch?

### Follow-up Questions
- Does `filter` guarantee the same array length as the input, the way `map` does?
- Is `find` guaranteed to return the first matching element, or could it return any match?

### Code Prediction
```javascript
const items = [{ id: 1, qty: 1 }, { id: 2, qty: 1 }];
const updated = items.map(item => item.id === 2 ? { ...item, qty: item.qty + 1 } : item);
console.log(items[1].qty, updated[1].qty);
console.log(items === updated);
```
Predict all three outputs, and explain why `items` itself is untouched despite `updated` reflecting the change.

## Practical Tasks

- Rewrite a manual `for` loop using `push()` to build a filtered/transformed array into an equivalent using `map`/`filter`.
- Implement an immutable "toggle one item's done flag" update for a list of todo objects.
- Replace a `.sort()` call on a shared array with `.toSorted()` and verify the original array is left unchanged.

## Readiness Criteria

Use `map`/`filter`/`reduce` fluently for common data transformations, implement immutable single-item updates within an array, and explain why mutating methods are risky specifically in UI-adjacent code.

## References

- [MDN: Array.prototype.map()](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
- [MDN: Array.prototype.reduce()](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)
- [MDN: Array.prototype.toSorted()](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array/toSorted)
