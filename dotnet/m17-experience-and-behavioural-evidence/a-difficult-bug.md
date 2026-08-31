# A Difficult Bug

## What Interviewers Are Listening For

Not whether you can fix bugs — everyone can, eventually. They're listening for your *process*: did you form a hypothesis and test it, or thrash randomly until something worked? Did you understand the actual root cause, or just make the symptom go away? This maps directly to Module 16's debugging-broken-code exercises — the same discipline, now described from memory instead of solved live.

## Prompts to Find Your Real Story

- Have you debugged a race condition, deadlock, or thread-pool starvation issue (Module 6)?
- Have you traced a bug to a closure capturing the wrong variable, or an `async void` swallowing an exception (Module 2/6)?
- Have you found a bug that only appeared under load or in production, never locally (Module 13's measure-before-optimizing content)?
- Have you diagnosed a silent model-binding failure or a mass-assignment issue (Module 8/12)?
- Was there a bug where your *first* hypothesis was wrong, and you had to revise it? (These often make the most credible stories — a bug solved on the first guess can sound too easy.)

## A Generic Shape (Template Only — Fill In Your Actual Details)

```
Situation: [what the symptom was, and why it mattered — e.g., affected users, blocked a release]
Task:      [what you were specifically asked to do, or took on]
Action:    [your actual hypothesis, how you tested it, what tool you used — a debugger,
            logging, a profiler — and what you found; if your first hypothesis was wrong,
            say so and explain what that ruled out]
Result:    [the fix, how you verified it, and ideally a regression test or monitoring change
            that would catch this specific bug if it ever recurred]
```

## Common Weak Answers

- Describing the bug's symptom in detail but glossing over the actual diagnostic process ("I looked into it and found the issue") with no specifics about *how*.
- No mention of verification — did you confirm the fix actually worked, or just assume it did?
- A story where the fix was "add a try/catch" or "add a null check" with no explanation of the actual root cause underneath the symptom.
- Taking sole credit for a fix that was genuinely a team effort, or conversely being so vague about your own contribution that it's unclear what you actually did.

## Follow-up Questions to Expect

- "How did you first notice something was wrong?"
- "What was your first theory, and was it right?"
- "How did you make sure it wouldn't happen again?"
- "What would you do differently if you hit a similar bug today?"

## References

- [Race conditions, deadlocks, and shared mutable state (Module 6)](../m06-async-concurrency/race-conditions-and-deadlocks.md)
- [Debugging broken code (Module 16)](../m16-practical-interview-exercises/debugging-broken-code.md)
- [Measuring before optimizing (Module 13)](../m13-performance-diagnostics-observability/measure-before-optimizing.md)
