# Code Review

## Definition

Code review is the practice of having another person examine a proposed change before it merges — catching bugs, correctness issues, and design problems a single author might miss, sharing knowledge across the team, and maintaining consistency in how the codebase evolves. It's a skill in both directions: giving useful, constructive feedback as a reviewer, and receiving it well as an author.

```
A good review comment: "This doesn't handle the case where `items` is empty — CalculateTotal
would throw here. Worth a guard clause?"
A poor review comment: "This is wrong."
```

## Alternatives & Trade-offs

Skipping review (merging without a second set of eyes) is faster in the moment but loses the specific value review provides: catching what the author's own familiarity with the change blinds them to, and spreading knowledge of the change across the team instead of leaving it siloed with one person. Review costs reviewer time and some latency before a change lands, in exchange for those benefits — the practical question is usually about calibrating review depth/speed to the change's actual risk, not whether to review at all.

## How It Works

### Reviewing for correctness, design, and readability — not just style

```
Correctness: does this actually do what it's supposed to, including edge cases?
Design:      does this fit the codebase's existing patterns and boundaries (Module 4/14's
              content), or does it introduce an inconsistency that will cause friction later?
Readability: will someone unfamiliar with this change understand it in six months?
```

Style (formatting, naming conventions) is real but is exactly the kind of concern automated tooling (static analysis, a later topic) should catch, freeing human review time for the things automation can't — correctness and design judgment.

### Giving feedback that's specific, actionable, and kind

```
Vague: "This seems off."
Specific and actionable: "If `customerId` is 0 here, GetCustomerAsync will return null and
                          this will NullReferenceException on the next line. Should this
                          validate customerId first, or handle a null result?"
```

Explaining *why* something is a concern, and ideally what to do about it, turns a review comment into something the author can actually act on, rather than something that just feels like criticism.

### Receiving feedback well — treating review as collaborative, not adversarial

```
A defensive response to review feedback ("well it works, doesn't it") tends to suppress
future feedback, including feedback that would have caught a real problem next time. Treating
review comments as genuinely useful information, even when the code technically "works,"
keeps the review process actually functional as a quality mechanism.
```

### Calibrating review depth to risk

```
A one-line typo fix in a comment: a quick glance is proportionate.
A change to the payment-processing critical path: deserves careful, thorough review,
proportional to the cost of a mistake reaching production there specifically.
```

Not every change needs the same depth of scrutiny — over-scrutinizing trivial changes wastes reviewer time that could go toward genuinely risky changes, while under-scrutinizing risky changes defeats the purpose of review entirely.

## Application

Review for correctness and design first, relying on automated tooling for style. Give feedback that's specific about the actual concern and, where possible, suggests a path forward. Receive feedback as useful information rather than criticism to defend against. Calibrate review depth to the actual risk of the change rather than applying uniform scrutiny everywhere.

## Common Mistakes

- Giving vague, non-actionable feedback ("this is wrong," "not a fan of this") that doesn't explain the actual concern or what to do about it.
- Reviewing for style/formatting concerns that automated tooling should be catching, instead of focusing human attention on correctness and design.
- Responding defensively to review feedback, which discourages reviewers from giving thorough feedback in the future.
- Applying the same shallow (or same exhaustive) level of scrutiny to every change regardless of its actual risk or complexity.

## Common Interview Questions

### Basic
- What is code review, and what value does it add beyond just "checking for bugs"?
- What makes review feedback actionable versus not?

### Intermediate
- Why should style/formatting concerns generally be handled by automated tooling rather than human review comments?
- How would you calibrate review depth based on a change's risk?

### Advanced
- How would you give critical feedback on a design decision in a way that's honest but doesn't feel personally adversarial to the author?
- How would you handle a situation where you disagree with a reviewer's feedback but believe your original approach is correct?

### Follow-up Questions
- Should every PR require the same number of approvals regardless of size or risk?
- Is it appropriate to approve a PR with minor unresolved comments, trusting the author to address them?

### Code Prediction
A reviewer leaves the comment "this is wrong" on a line of code with no further explanation. What information does the author actually have to act on, compared to a comment explaining the specific scenario that breaks and why?

## Practical Tasks

- Review a sample pull request, writing specific, actionable feedback for at least one correctness concern and one design concern.
- Practice responding constructively to a piece of critical feedback on a hypothetical change you authored.
- Calibrate review depth for three hypothetical changes of varying risk (a typo fix, a new feature, a payment-processing change).

## Readiness Criteria

Give specific, actionable code review feedback focused on correctness and design, receive feedback collaboratively rather than defensively, and calibrate review depth to actual risk.

## References

### Other

- [Google Engineering Practices: How to do a code review](https://google.github.io/eng-practices/review/reviewer/)
