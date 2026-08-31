# Improving Performance

## What Interviewers Are Listening For

Whether you measured before optimizing (Module 13's core discipline) rather than guessing, and whether you can quantify the actual improvement — "it's faster now" is a much weaker answer than a specific before/after number.

## Prompts to Find Your Real Story

- Did you fix an N+1 query problem (Module 10/16), a missing index (Module 9), or unnecessary allocations (Module 13) using an actual profiler or query log, not intuition?
- Did you introduce caching (Module 8/13/14) for a genuinely read-heavy path, and how did you decide it was actually the right fix?
- Did you diagnose a CPU-bound vs. I/O-bound bottleneck (Module 6/13) and apply the correct category of fix?
- Was there a performance fix that turned out to be the wrong one initially, requiring you to revise your approach after measuring again?

## A Generic Shape (Template Only — Fill In Your Actual Details)

```
Situation: [what was slow, and how you first learned about it — a user complaint, a metric
            alert (Module 13), your own investigation]
Task:      [what target you were working toward — a specific latency goal, a specific load
            level the system needed to handle]
Action:    [HOW you found the actual bottleneck — a profiler, query logging, a load test
            (Module 13) — before making any change; then what you actually changed]
Result:    [the measured before/after — a specific number, not just "much faster" — and how
            you verified it held up under real load, not just in a quick local test]
```

## Common Weak Answers

- No measurement at all, either before diagnosing the problem or after fixing it — "it felt faster" instead of an actual number.
- Applying a fix you'd read about (add caching, add an index) without first confirming that was actually the bottleneck for this specific case.
- Confusing correlation with causation — attributing an improvement to your change when something else (reduced traffic, a different deploy) might explain it, without acknowledging that ambiguity if it's genuinely present.
- Choosing an example so small or artificial that it doesn't demonstrate a real investigative process.

## Follow-up Questions to Expect

- "How did you know where the actual bottleneck was, before you started?"
- "What were the before and after numbers, specifically?"
- "Did the fix hold up under real production load, not just your test?"

## References

- [Measuring before optimizing (Module 13)](../m13-performance-diagnostics-observability/measure-before-optimizing.md)
- [N+1 queries (Module 10)](../m10-entity-framework-core/n-plus-one-queries.md)
- [Caching strategies: in-memory vs. distributed (Module 13)](../m13-performance-diagnostics-observability/caching-strategies.md)
- [Finding EF Core performance issues (Module 16)](../m16-practical-interview-exercises/finding-ef-core-performance-issues.md)
