# Working with Unclear Requirements

## What Interviewers Are Listening For

Whether you ask clarifying questions and make a reasonable, stated assumption when you can't get an answer immediately — versus either freezing, or guessing silently and building the wrong thing. This is genuinely common in real work, and a story with no ambiguity at all usually isn't a realistic one.

## Prompts to Find Your Real Story

- Was there a feature request where the actual business rule was ambiguous — e.g., what should happen at a specific edge case (a discount at exactly 100%, an order with zero items) — and you had to decide how to interpret it?
- Did you build something, get feedback that it wasn't quite what was wanted, and have to adjust — and what did you learn about clarifying earlier next time?
- Was there a case where you made a reasonable assumption, stated it explicitly (in a PR description, a message to a stakeholder), and it turned out to be right — or wrong?
- Did unclear requirements ever reveal that the requester themselves hadn't fully thought through what they wanted, and how did you help surface that?

## A Generic Shape (Template Only — Fill In Your Actual Details)

```
Situation: [what was ambiguous, specifically — not just "requirements were unclear" but
            what exactly was underspecified]
Task:      [what you needed to deliver despite the ambiguity, and any real time pressure]
Action:    [did you ask a clarifying question, and if the answer wasn't immediately
            available, what assumption did you make and how did you communicate it?]
Result:    [whether your assumption was right, what happened if it wasn't, and what you'd
            do differently with similarly unclear requirements next time]
```

## Common Weak Answers

- A story where you simply guessed silently with no communication of the assumption you were making, even if the guess happened to be right — the communication is the actual skill being tested, not just luck.
- Framing all ambiguity as someone else's failure to specify clearly, rather than acknowledging that some ambiguity is normal and part of the job to navigate.
- No account of what happened when (or if) your assumption turned out to be wrong — a story where everything went perfectly on the first guess can seem incomplete or convenient.
- Excessive question-asking with no willingness to make any reasonable assumption at all, which can itself be a weakness if taken to an extreme.

## Follow-up Questions to Expect

- "How did you decide when to just ask versus when to make an assumption and move forward?"
- "What did you do if the person you needed clarification from wasn't available?"
- "Was your assumption ever wrong, and what happened then?"

## References

This scenario is less mapped to a specific technical module and more about communication discipline — Module 8's validation content and Module 7's API-design content are useful anchors if your example involves interpreting an ambiguous business rule into concrete code.
