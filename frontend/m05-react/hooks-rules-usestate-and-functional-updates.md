# Hooks, `useState`, and Functional Updates

## Definition

Hooks are functions that let a function component use React features such as state and lifecycle synchronization. `useState` stores component state across renders. A functional update (`setState(prev => next)`) computes the next value from the most recently queued state, rather than from whatever value was captured when the update was scheduled.

```tsx
const [count, setCount] = useState(0);

function handleClick() {
  setCount(count + 1); // uses the value CAPTURED by this render's closure
}
```

## Alternatives & Trade-offs

Reading `count` directly (`setCount(count + 1)`) is simpler to write and correct as long as only one update is ever queued per render. A functional update (`setCount(c => c + 1)`) is slightly more ceremony, but is the only reliable choice when more than one state update might be queued from the same event or when the update genuinely depends on the latest, not-yet-rendered value — the exact situation the next section demonstrates going wrong.

## How It Works

### The Rules of Hooks — why call order must stay identical every render

```tsx
// WRONG: a hook called conditionally changes how many hooks run on different renders
function Profile({ userId }: { userId: string }) {
  if (userId) {
    const [name, setName] = useState(""); // React can no longer reliably match this to the right slot
  }
  const [email, setEmail] = useState("");
}

// RIGHT: hooks always called in the same order; put the condition INSIDE the hook's logic instead
function Profile({ userId }: { userId: string }) {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  if (!userId) return null;
}
```

React associates each hook call with a slot based purely on *call order*, not name or content — calling a hook conditionally shifts every subsequent hook's slot on renders where the condition differs, silently corrupting which piece of state belongs to which `useState` call.

### Why three direct updates in one handler don't produce three increments

```tsx
const [count, setCount] = useState(0);

function handleTripleClick() {
  setCount(count + 1); // count is 0 (this render's captured value) -> queues "set to 1"
  setCount(count + 1); // count is STILL 0 in this closure -> queues "set to 1" again
  setCount(count + 1); // still 0 -> queues "set to 1" again
}
// after handleTripleClick runs once: count becomes 1, NOT 3
```

Each call reads the exact same `count` variable — the value captured by this render's closure — because `count` doesn't change until the next render actually happens. All three calls compute the same "next value," so only the last one queued effectively matters.

### The functional-update fix — each update builds on the previous queued one

```tsx
function handleTripleClickFixed() {
  setCount(c => c + 1); // queues: take whatever the state ends up being, add 1
  setCount(c => c + 1); // queues: take the RESULT of the previous queued update, add 1
  setCount(c => c + 1); // queues: take THAT result, add 1
}
// after handleTripleClickFixed runs once: count becomes 3
```

A functional update receives the most recently *queued* value, not the value captured at render time — this is what allows several updates in the same event to compose correctly instead of each overwriting the others' intent.

### State updates don't change the current render's own variable

```tsx
function handleClick() {
  setCount(count + 1);
  console.log(count); // still logs the OLD value — this render's `count` never changes after the fact
}
```

`setCount` schedules a future render with a new value; it does not mutate `count` in the render that called it. Reading `count` again immediately afterward, in the same function call, still sees the value this render started with.

## Application

Use a functional update whenever more than one state change might be queued in the same event, or whenever the next value genuinely depends on the latest state rather than a value already known at render time. Never call a hook conditionally, in a loop, or after an early return — hook calls must execute in the exact same order on every render.

## Common Mistakes

- Calling a hook conditionally or after an early return, changing hook call order between renders and corrupting state association.
- Chaining several `setState(value + 1)` calls in one handler, expecting each to build on the previous one, when all three read the same stale closure value.
- Expecting a state variable to reflect its new value immediately after calling its setter, within the same function call.
- Reaching for `useReducer` or extra state variables to work around a bug that a single functional update would have fixed directly.

## Common Interview Questions

### Basic
- What is a React hook, and what is `useState` used for?
- Why must hooks always be called in the same order on every render?

### Intermediate
- Why does calling `setCount(count + 1)` three times in a row only increment the count by one?
- When does a functional update behave differently from passing the new value directly?

### Advanced
- Walk through, step by step, why `setCount(c => c + 1)` called three times produces three increments while `setCount(count + 1)` called three times does not.
- What actually breaks if a `useState` call is moved inside an `if` block that's sometimes true and sometimes false?

### Follow-up Questions
- Does calling a state setter cause the component to rerender synchronously, before the next line of code runs?
- Is it ever correct to call `setCount(count + 1)` more than once in the same handler?

### Code Prediction
```tsx
const [count, setCount] = useState(0);
function handleClick() {
  setCount(count + 1);
  setCount(count + 1);
  setCount(c => c + 1);
}
```
Given `count` starts at `0`, what value does `count` end up at after one click, and why does the third call behave differently from the first two?

## Practical Tasks

- Reproduce the triple-`setCount(count + 1)` bug and fix it using functional updates.
- Refactor a component with a conditionally-called `useState` hook into one that calls every hook unconditionally.
- Write a counter component supporting a "reset to zero, then increment three times" button using functional updates throughout.

## Readiness Criteria

Explain the Rules of Hooks and why violating them corrupts state, and predict the difference between direct and functional state updates precisely, including when multiple updates are queued in one handler.

## References

- [React: Rules of Hooks](https://react.dev/reference/rules/rules-of-hooks)
- [React: `useState`](https://react.dev/reference/react/useState)
- [React: Queueing a series of state updates](https://react.dev/learn/queueing-a-series-of-state-updates)
