# Making a Design Trade-off

## What Interviewers Are Listening For

Whether you can articulate *why* you chose one option over another — including the cost of your choice, not just its benefit. This is the behavioral mirror of Module 14's entire architectural framing: every pattern earns its cost only when a real requirement justifies it, and a strong answer here demonstrates that judgment applied to a real decision you made.

## Prompts to Find Your Real Story

- Did you decide whether to introduce a repository abstraction, a caching layer, or a more sophisticated architecture (Module 4/14) — and can you state the actual trade-off, not just the choice?
- Did you choose between normalizing and denormalizing a piece of data (Module 9), or between synchronous and asynchronous communication (Module 14)?
- Was there a trade-off between shipping something simpler now versus something more "correct" but slower to build — and how did you decide?
- Did you make a trade-off that, in hindsight, you'd make differently, and why?

## A Generic Shape (Template Only — Fill In Your Actual Details)

```
Situation: [what decision needed to be made, and what the real options were]
Task:      [what constraint made this a genuine trade-off rather than an obvious choice —
            time, complexity, a requirement that pulled against simplicity]
Action:    [what you actually chose, and — this is the important part — what you gave up
            by choosing it, stated explicitly]
Result:    [how it played out, and whether the trade-off still looks right in hindsight]
```

## Common Weak Answers

- Describing the chosen option's benefits at length while never naming what was given up — a real trade-off has a real cost, and not naming it suggests either the decision wasn't well-understood or is being retold too simply.
- Presenting the decision as obvious in hindsight when it likely wasn't at the time, understating the actual judgment involved.
- Choosing an example with no real alternative that was seriously considered, which isn't really a trade-off story at all.
- Not being willing to say the trade-off might look different in hindsight, when that's often genuinely true and shows real reflection.

## Follow-up Questions to Expect

- "What did you give up by choosing that option?"
- "What would have made you choose the other option instead?"
- "Would you make the same choice again today?"

## References

- [Avoiding unnecessary architecture (Module 14)](../m14-architecture-and-system-design/avoiding-unnecessary-architecture.md)
- [Normalization and denormalization trade-offs (Module 9)](../m09-relational-databases-and-sql/normalization-and-denormalization.md)
- [Repository pattern (Module 4)](../m04-oop-design/repository.md)
