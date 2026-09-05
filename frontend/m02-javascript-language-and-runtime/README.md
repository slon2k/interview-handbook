# Module 2 - JavaScript Language and Runtime

**Status:** Complete  
**Priority:** Critical  
**Prerequisites:** [Module 1 - Web Platform Foundations](../m01-web-platform-foundations/README.md)

## Scope

This module establishes the JavaScript mental models that sit underneath TypeScript and React. It focuses on runtime behaviour: what values are, how references and mutation work, when bindings exist, how functions capture state, and why asynchronous code executes in a particular order.

The goal is not syntax recall. A strong frontend candidate can predict a small program, explain an unexpected UI bug, and choose a clear, maintainable expression of the required behaviour.

## Learning Outcomes

By the end of this module, you should be able to:

- Distinguish primitive values from object references and predict mutation, copying, and equality behaviour.
- Use objects and arrays without accidental shared state or mutation.
- Explain coercion, truthiness, `null`, and `undefined`, and use strict equality deliberately.
- Predict scope, hoisting, temporal-dead-zone, closure, and `this` behaviour.
- Use modern JavaScript syntax and modules to transform data clearly and safely.
- Handle errors, promises, and `async`/`await` without losing failures or creating accidental sequential work.
- Explain the browser event loop, microtasks, and macrotasks well enough to predict common ordering questions.
- Recognise prototype, class, iterator, generator, and garbage-collection concepts at working-awareness level.

## Topics

### 1. Values and Data Structures

- [Values, types, and equality](values-types-and-equality.md)
- [Objects, arrays, references, and mutation](objects-arrays-references-and-mutation.md)
- [Coercion, truthiness, null, and undefined](coercion-truthiness-null-and-undefined.md)

### 2. Bindings and Functions

- [Scope, declarations, hoisting, and the temporal dead zone](scope-declarations-hoisting-and-temporal-dead-zone.md)
- [Functions, arrow functions, and this](functions-arrow-functions-and-this.md)
- [Closures](closures.md)

### 3. Modern Everyday JavaScript

- [Destructuring, spread, rest, optional chaining, and nullish coalescing](modern-javascript-syntax.md)
- [Array methods and immutable data transformations](array-methods-and-immutable-data-transformations.md)
- [Modules and error handling](modules-and-error-handling.md)

### 4. Asynchronous JavaScript and Runtime Awareness

- [Promises and async/await](promises-and-async-await.md)
- [The event loop, microtasks, and macrotasks](event-loop-microtasks-and-macrotasks.md)
- [Prototypes, classes, iterables, generators, and garbage collection](runtime-awareness-prototypes-iterables-and-garbage-collection.md)

## Scope Boundaries

- Type annotations, narrowing, generics, and compiler configuration belong in Module 3 - TypeScript. JavaScript runtime behaviour remains relevant even in a TypeScript project.
- DOM events, `fetch`, browser storage, rendering, and API integration belong in Module 4 - Browser Platform and ASP.NET Core API Integration.
- React rendering, state, effects, and React-specific stale closures belong in Module 5 - React; this module establishes the JavaScript rules underneath them.
- Test runners and frontend build tooling belong in Module 6 - Frontend Testing and Tooling.

## Suggested Learning Sequence

1. Learn value categories, object references, mutation, equality, and coercion before writing data transformations.
2. Learn scope and declarations, then functions, `this`, and closures.
3. Practise modern syntax, modules, error handling, and array transformations on realistic UI data.
4. Learn promises and `async`/`await`, then use the event loop to explain observable execution order.
5. Finish with prototype, class, iterator, generator, and garbage-collection awareness.

## Practical Deliverables

- Predict and explain object mutation, equality, scope, closure, and event-loop snippets without running them.
- Transform an API result into display data without mutating the original value.
- Repair a callback that captures the wrong state or loses its `this` binding.
- Refactor an error-prone promise chain into `async`/`await` with deliberate error handling.
- Explain why two independent requests are accidentally sequential and how to start them concurrently.

## Interview Coverage

Each topic includes foundation questions, everyday trade-offs, advanced follow-ups, and a code-prediction prompt. Practise explaining the execution model in plain language before reaching for framework-specific terminology.

## References

- [MDN: JavaScript Guide](https://developer.mozilla.org/docs/Web/JavaScript/Guide)
- [MDN: JavaScript reference](https://developer.mozilla.org/docs/Web/JavaScript/Reference)