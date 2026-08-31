# Handling Technical Debt

## What Interviewers Are Listening For

Whether you can identify technical debt precisely (not just "the code was bad"), and whether you can make a case for addressing it in terms a non-technical stakeholder would find persuasive — or, just as importantly, whether you know when *not* to prioritize it. This connects to Module 14's avoiding-unnecessary-architecture theme from the other direction: recognizing when complexity/shortcuts genuinely need addressing versus when they're tolerable.

## Prompts to Find Your Real Story

- Did you identify a specific piece of debt — a missing test suite (Module 11), a tangled module boundary (Module 4/14), an outdated dependency with a known vulnerability (Module 15) — and make the case to actually address it?
- Did you have to argue for spending time on debt against pressure to ship new features, and how did you frame the argument?
- Did you deliberately take on debt yourself, knowingly, to hit a deadline — and how did you make sure it didn't get forgotten afterward?
- Was there debt you decided *not* to address, and can you explain why that was the right call at the time?

## A Generic Shape (Template Only — Fill In Your Actual Details)

```
Situation: [what the debt was, specifically, and what risk or cost it was actually
            creating — not just "it was messy" but a concrete consequence]
Task:      [what you were trying to accomplish — get buy-in to address it, or decide
            whether it was worth addressing at all]
Action:    [how you made the case — did you frame it in terms of risk, velocity, or a
            specific incident it had already caused or was likely to cause?]
Result:    [whether it got addressed, what happened as a result, or — if it deliberately
            didn't get addressed — why that was the right call]
```

## Common Weak Answers

- Vague framing ("the codebase had a lot of technical debt") with no specific, named example of what that debt actually was or what risk it created.
- A story with no attempt to make a business case — technical debt arguments that only speak in technical terms often fail to get prioritized, and a strong answer shows awareness of that.
- Treating all debt as equally urgent, missing the judgment call (Module 14's theme) of recognizing which debt is actually costly versus which is tolerable.
- No example of ever *not* prioritizing debt when it wasn't actually worth the cost — a story that always ends with "and we fixed it" can seem like it's missing the harder, more common case where debt is knowingly deferred.

## Follow-up Questions to Expect

- "How did you decide this specific debt was worth prioritizing over new feature work?"
- "How did you frame the argument to someone who might not care about the technical details?"
- "Is there debt you're aware of right now that you've deliberately chosen not to address? Why?"

## References

- [Code smells (Module 4)](../m04-oop-design/code-smells.md)
- [Avoiding unnecessary architecture (Module 14)](../m14-architecture-and-system-design/avoiding-unnecessary-architecture.md)
- [NuGet dependency management (Module 15)](../m15-development-workflow-and-delivery/nuget-dependency-management.md)
