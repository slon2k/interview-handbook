# Prototypes, Classes, Iterables, Generators, and Garbage Collection

## Definition

JavaScript objects inherit through prototype chains. `class` syntax provides a familiar declaration form over that model. Iterables provide values to `for...of`, spread, and other consumers; generators create iterable sequences lazily. Garbage collection reclaims unreachable memory automatically.

## How It Works

- Property lookup checks an object, then its prototype chain until it finds a property or reaches `null`.
- A JavaScript `class` creates constructor and prototype methods; it is not the same runtime model as C# classes.
- Arrays, strings, maps, and sets are iterable. Plain objects are not iterable by default.
- A generator function returns an iterator whose `yield` values are produced on demand.
- Garbage collectors reclaim objects that are no longer reachable, but event listeners, timers, caches, and closures can keep values reachable longer than intended.

## Application

For most React application work, use plain objects, arrays, and functions. Recognise class and prototype behaviour when reading libraries or legacy code. Prevent leaks by removing listeners, clearing timers, and defining cache lifetimes rather than attempting to force garbage collection.

## Common Mistakes

- Assuming classes remove or replace prototypes.
- Using `for...of` directly on a plain object.
- Assuming garbage collection immediately frees an unreachable object.
- Blaming garbage collection for memory retained by an active listener or cache.

## Common Interview Questions

### Foundation

- What is a prototype chain?
- Which built-in values are iterable?

### Intermediate

- How does `class` relate to prototypes?
- What makes a generator useful for a large or deferred sequence?

### Advanced and Follow-up

- How can a listener or closure cause memory to remain reachable?

### Code Prediction

Predict whether `for...of` works for an array and a plain object, and where a property lookup proceeds after it is absent from an object itself.

## Practical Tasks

- Inspect a class instance and identify a property stored on the instance versus a method on its prototype.
- Review a feature for listeners or timers that must be cleaned up when it is no longer needed.

## Readiness Criteria

You can recognise the JavaScript object model, use iterables appropriately, and reason about reachability-based memory management at awareness level.

## References

- [MDN: Inheritance and the prototype chain](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain)
- [MDN: Iteration protocols](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Iteration_protocols)
- [MDN: Memory management](https://developer.mozilla.org/docs/Web/JavaScript/Memory_management)