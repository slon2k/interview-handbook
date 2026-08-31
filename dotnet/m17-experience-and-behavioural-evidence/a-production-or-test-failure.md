# A Production or Test Failure

## What Interviewers Are Listening For

How you behave under pressure and whether you focus on stabilizing and learning rather than blame. A weak answer here is a serious red flag precisely because production incidents are stressful and revealing — interviewers are gauging whether you'd be someone they'd trust to be calm and useful during a real one.

## Prompts to Find Your Real Story

- Was there a deployment that broke something in production — a bad migration (Module 10/15), a missing feature flag guard (Module 15), a config error (Module 8/15)?
- Was there a flaky or failing test that revealed a real bug rather than being "just flaky" (Module 11's reliable-and-deterministic-tests content)?
- Was there an incident where the root cause turned out to be something surprising — a race condition only appearing under load (Module 6/13), a cache invalidation gap (Module 13/14)?
- Was there a time your own change caused the failure? (A story where you take ownership of your own mistake, calmly and specifically, is often stronger than one where someone else's mistake is centered.)

## A Generic Shape (Template Only — Fill In Your Actual Details)

```
Situation: [what broke, what the user/business impact was, roughly how it was detected —
            an alert, a health check (Module 8/13), a user report]
Task:      [your specific role in the response — were you the one who noticed it, fixed it,
            communicated about it?]
Action:    [what you did to stabilize FIRST — a rollback (Module 15), a feature flag flip
            (Module 15), a hotfix — separate from what you did to find the root cause after]
Result:    [how it was resolved, the root cause, and what changed afterward — a new test,
            a new alert, a process change]
```

## Common Weak Answers

- Framing the incident entirely around blaming a teammate, a vague "process failure," or "bad luck," with no ownership of your own part.
- Conflating stabilizing the immediate problem with fixing the root cause — a strong answer distinguishes "first we rolled back" from "then we found out why," since real incidents usually require both, in that order.
- No concrete follow-up action — if nothing changed as a result of the incident (a new test, a new alert, a new process), that's worth noticing and addressing honestly rather than glossing over.
- Describing panic or chaos with no indication you brought or helped bring calm to the situation.

## Follow-up Questions to Expect

- "What did you do in the first five minutes?"
- "How did you communicate what was happening to others?"
- "What changed afterward so this specific failure mode couldn't happen again?"
- "Looking back, what's the earliest point you could have caught this?"

## References

- [Basic release and rollback thinking (Module 15)](../m15-development-workflow-and-delivery/release-and-rollback-thinking.md)
- [Feature flags (Module 15)](../m15-development-workflow-and-delivery/feature-flags.md)
- [Reliable and deterministic tests (Module 11)](../m11-testing-and-testability/reliable-and-deterministic-tests.md)
- [Health checks in a monitoring and observability context (Module 13)](../m13-performance-diagnostics-observability/health-checks-and-monitoring.md)
