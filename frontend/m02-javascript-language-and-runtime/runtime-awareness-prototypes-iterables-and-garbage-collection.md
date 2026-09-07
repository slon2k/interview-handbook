# Prototypes, Classes, Iterables, Generators, and Garbage Collection

## Definition

This is working-awareness content: JavaScript's object model is prototype-based underneath (`class` syntax is sugar over it), iterables/iterators power `for...of` and spread, generators produce values lazily one at a time, and garbage collection reclaims memory for objects no longer reachable. None of these need daily hands-on mastery for typical React work, but each explains behavior you'll encounter.

```javascript
class Animal {
  constructor(name) { this.name = name; }
  speak() { return `${this.name} makes a sound`; }
}
const dog = new Animal("Rex");
console.log(Object.getPrototypeOf(dog) === Animal.prototype); // true — class methods live on the prototype, not each instance
```

## Alternatives & Trade-offs

`class` syntax reads more familiarly to developers coming from other object-oriented languages and is the standard, recommended way to define object blueprints in modern JavaScript. Working directly with prototypes (`Object.create`, manually assigning to `.prototype`) is more explicit about what's actually happening underneath, but more verbose and rarely written by hand in modern code — understanding it mainly serves to explain what `class` is doing on your behalf.

## How It Works

### Prototypes — how method lookup actually works

```javascript
class Animal {
  constructor(name) { this.name = name; }
  speak() { return `${this.name} makes a sound`; }
}

const dog = new Animal("Rex");
dog.speak();                                     // JavaScript looks for `speak` on `dog` itself — not found —
                                                     // then walks UP to Animal.prototype, where it IS found

Object.getPrototypeOf(dog) === Animal.prototype;    // true
dog.hasOwnProperty("name");                            // true — name is an OWN property, set in the constructor
dog.hasOwnProperty("speak");                              // false — speak lives on the prototype, shared across ALL instances
```

Every instance of `Animal` shares the *same* `speak` function via the prototype chain, rather than each instance carrying its own independent copy — this is why methods defined in a `class` body are memory-efficient regardless of how many instances are created.

### Iterables and `for...of` — what makes something loopable

```javascript
const numbers = [1, 2, 3];
for (const n of numbers) { console.log(n); } // works — arrays are iterable

const plainObject = { a: 1, b: 2 };
for (const x of plainObject) { } // TypeError: plainObject is not iterable — plain objects are NOT iterable by default
```

An object is iterable if it implements the well-known `Symbol.iterator` method — arrays, strings, `Map`, and `Set` all do this built in; a plain object doesn't, which is exactly why `for...of` (and spread, `[...obj]`) fail on plain objects but succeed on arrays.

### Generators — functions that produce values lazily, one at a time

```javascript
function* countUpTo(max) {
  for (let i = 1; i <= max; i++) {
    yield i; // pauses HERE, returning i, until the next value is requested
  }
}

const counter = countUpTo(3);
console.log(counter.next()); // { value: 1, done: false }
console.log(counter.next()); // { value: 2, done: false }
console.log(counter.next()); // { value: 3, done: false }
console.log(counter.next()); // { value: undefined, done: true }

for (const n of countUpTo(3)) { console.log(n); } // 1, 2, 3 — generators are ALSO iterables, usable with for...of
```

Unlike a regular function, which runs completely and returns once, a generator's execution pauses at each `yield` and resumes only when the next value is explicitly requested — useful for producing values lazily (an infinite sequence, or a large dataset processed one chunk at a time) without computing everything upfront.

### Garbage collection — reachability, not reference counting

```javascript
let user = { name: "Alice" };
user = null; // the ORIGINAL object is now unreachable (assuming nothing else references it) —
              // eligible for garbage collection, though exactly WHEN it's collected is not specified or guaranteed
```

```javascript
function attachHandler() {
  const largeData = new Array(1_000_000).fill("x");
  document.addEventListener("click", () => console.log(largeData.length));
  // largeData stays reachable (and therefore NOT garbage collected) for as long as this
  // listener exists, since the closure keeps a live reference to it — the closures topic's
  // memory-retention concern, from the garbage collector's actual perspective
}
```

Modern JavaScript engines use reachability-based garbage collection: an object is collected once nothing reachable from a "root" (global scope, the current call stack, or an active closure) still references it — not based on counting how many references currently point to it, which is a different, older strategy some other languages use.

## Application

Recognize that `class` methods live on the prototype and are shared across instances, not duplicated per object. Know that `for...of` and spread require an object to be iterable, which plain objects aren't by default. Use generators specifically for lazy, one-value-at-a-time production where computing an entire collection upfront would be wasteful. Understand garbage collection well enough to reason about why a long-lived closure or event listener can keep otherwise-unneeded data alive.

## Common Mistakes

- Assuming each class instance has its own independent copy of every method, rather than sharing them via the prototype chain.
- Trying to use `for...of` or spread on a plain object, forgetting it isn't iterable unless it implements `Symbol.iterator`.
- Confusing a generator function's lazy, pause-and-resume execution with a regular function that runs to completion in one go.
- Assuming an object is garbage collected the instant a variable pointing to it is set to `null`, when reachability from *any* other live reference (including inside a closure) keeps it alive regardless.

## Common Interview Questions

### Basic
- Is JavaScript's `class` syntax a fundamentally different object model from prototypes, or built on top of them?
- What makes an object "iterable"?

### Intermediate
- Why can't you use `for...of` directly on a plain object?
- What's the practical difference between a generator function and a regular function?

### Advanced
- Walk through how method lookup traverses the prototype chain when a method isn't found directly on an instance.
- How does a closure retained by a long-lived event listener affect what the garbage collector can and can't reclaim?

### Follow-up Questions
- Do all instances of a class share the exact same function object for a given method, or does each get its own copy?
- Can a generator be paused indefinitely, or must it eventually be exhausted?

### Code Prediction
```javascript
class Counter {
  count = 0;
  increment() { this.count++; }
}
const a = new Counter();
const b = new Counter();
console.log(a.increment === b.increment);
```
Predict the output, and explain what it reveals about how class methods are shared (or not) across instances.

## Practical Tasks

- Inspect a class instance's prototype chain using `Object.getPrototypeOf` and `hasOwnProperty` to distinguish own properties from inherited methods.
- Implement a generator function that lazily produces an infinite sequence, consuming only a few values from it with `.next()`.
- Explain, for a code review, why an event listener capturing a large object in its closure could delay that object's garbage collection.

## Readiness Criteria

Explain prototype-based method sharing accurately, recognize iterable requirements for `for...of`/spread, use generators for lazy value production, and reason about reachability-based garbage collection in the context of closures.

## References

- [MDN: Object prototypes](https://developer.mozilla.org/docs/Learn/JavaScript/Objects/Object_prototypes)
- [MDN: Iterators and generators](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Iterators_and_generators)
- [MDN: Memory management](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Memory_management)
