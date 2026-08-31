# Refactoring Difficult Code

## What Interviewers Are Listening For

Whether you can improve code safely and incrementally rather than rewriting it wholesale and hoping — this maps directly to Module 4's refactoring-techniques and code-smells content, and Module 16's refactoring-tasks exercises, now told as a real story about a specific piece of legacy code.

## Prompts to Find Your Real Story

- Was there a long method, a "God class," or deeply tangled code you had to change (Module 4's code smells)?
- Did you refactor code with no existing tests, and how did you make that safe (a characterization test first, per Module 11)?
- Was there a refactor you did incrementally, in small steps, versus one you attempted all at once — and what did that teach you either way?
- Did you refactor something and discover, partway through, that your plan needed to change?

## A Generic Shape (Template Only — Fill In Your Actual Details)

```
Situation: [what made the code difficult — tangled responsibilities, no tests, unclear
            intent — and why it needed to change at all, not just "it was messy"]
Task:      [what you were trying to achieve — a new feature that the existing structure
            made hard, a bug that kept recurring because of the design, a performance issue]
Action:    [your actual approach — did you add tests first? Did you refactor in small,
            separately-committed steps (Module 15's commit-quality content)? How did you
            verify behavior didn't change along the way?]
Result:    [the concrete improvement — easier to extend, a bug class eliminated, faster
            onboarding for new team members — and whether it held up over time]
```

## Common Weak Answers

- Describing the "after" state in detail but glossing over the actual refactoring process and how you kept it safe.
- A refactor with no verification step at all — if you can't say how you confirmed behavior didn't change, that's a real gap worth being honest about, not glossing over.
- Choosing an example that was really a rewrite (throwing out the old code entirely) when asked specifically about refactoring, which are different activities with different risk profiles.
- No mention of testing at all, missing what's usually the single most important enabler of safe refactoring.

## Follow-up Questions to Expect

- "How did you make sure you didn't break anything?"
- "Did you do this in one big change or several smaller ones?"
- "What would have happened if there'd been no time to do this properly?"

## References

- [Code smells (Module 4)](../m04-oop-design/code-smells.md)
- [Refactoring techniques (Module 4)](../m04-oop-design/refactoring-techniques.md)
- [Refactoring tasks (Module 16)](../m16-practical-interview-exercises/refactoring-tasks.md)
- [Commit quality (Module 15)](../m15-development-workflow-and-delivery/commit-quality.md)
