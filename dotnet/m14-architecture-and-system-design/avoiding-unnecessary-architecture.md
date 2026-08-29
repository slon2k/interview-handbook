# Avoiding Unnecessary Architecture

## Definition

This closing topic names the theme running through this entire module explicitly: every pattern covered here — hexagonal architecture, CQRS, microservices, event-driven messaging, DDD's full process — earns its cost only when a real, current requirement justifies it. Applied without that justification, each one adds genuine complexity (more indirection, more moving parts, more concepts a new team member must learn) for no corresponding benefit — the architecture-scale version of Module 4's `pattern-overuse-and-overengineering.md`.

```
The question for every pattern in this module: "what specific, CURRENT requirement does this
solve that a simpler approach couldn't?" — not "could this be useful someday" or "is this what
a more sophisticated system would do."
```

## Alternatives & Trade-offs

Applying sophisticated architecture preemptively feels like responsible foresight, but the actual trade-off is concrete and immediate: more code, more concepts, slower onboarding, and more places for bugs to hide — paid *now*, in exchange for flexibility that may never actually be needed, or that could have been added later when the real requirement (if it ever materializes) is actually known and understood. Starting simpler and evolving toward complexity *when a real requirement demands it* defers that cost until it's actually justified, and often means the eventual solution fits the real requirement better than a speculative one guessed at upfront.

## How It Works

### A checklist for evaluating any pattern from this module before adopting it

```
1. What SPECIFIC requirement (not a hypothetical future one) does this solve?
2. What's the SIMPLEST thing that would also solve it?
3. What does adopting the more sophisticated option cost — in code, in concepts the team must
   learn, in operational complexity — RIGHT NOW?
4. Is the benefit large enough, TODAY, to justify that cost TODAY?
```

Applying this checklist to microservices: "we might need to scale independently someday" fails step 1 (not a current requirement); "our checkout process must handle 50x the load of everything else, starting next quarter" passes it.

### The concrete, cumulative cost of over-applying this module's patterns

```
A small team building a straightforward CRUD application that adopts full hexagonal
architecture + CQRS with separate read/write stores + event-driven microservices + full DDD
process, none of it justified by an actual current requirement, has made every single change —
even a trivial one — require touching far more code and understanding far more concepts than
the problem actually warranted.
```

None of these patterns is wrong in the abstract — each solves a real problem this module described — but stacking all of them without cause multiplies unnecessary cost across the whole system, not just once.

### Evolving toward complexity when a real requirement actually appears

```
Start: a modular monolith, one database, synchronous calls, one shared model for reads/writes.
A real requirement appears: the reporting dashboard's queries are now measurably too slow
against the write-optimized model.
Evolve: introduce the SIMPLE form of CQRS (Module 14's earlier topic) for just that read path —
        not a full architectural rewrite, just the smallest change addressing the actual problem.
```

This is the practical shape "start simple, evolve when justified" actually takes — not a single big rewrite, but incremental adoption of exactly the pattern a specific, now-real requirement calls for.

## Application

Before adopting any pattern from this module (or elsewhere in this handbook), apply the four-question checklist above. Default to the simplest approach that satisfies today's actual, demonstrated requirements. Treat sophisticated architecture as something to grow into deliberately, driven by a specific need that has actually appeared, not something to adopt speculatively because a more advanced system "would" have it.

## Common Mistakes

- Adopting a sophisticated pattern because it's associated with "good architecture" or "how serious systems are built," without a specific current requirement justifying it.
- Justifying complexity with a hypothetical future need ("we might need to scale independently someday") rather than a demonstrated current one.
- Adopting several sophisticated patterns simultaneously for a system that needed none of them yet, multiplying unnecessary cost across the whole codebase at once.
- Treating "starting simple" as a permanent constraint rather than a deliberate starting point meant to evolve when a real requirement actually appears.

## Common Interview Questions

### Basic
- Why might adopting a sophisticated architectural pattern without a current justifying requirement be a mistake?
- What's the difference between a hypothetical future need and a demonstrated current one, in this context?

### Intermediate
- How would you evaluate whether a proposed architectural pattern is actually justified for a given system right now?
- What's the practical cost of adopting an unjustified architectural pattern, beyond just "extra code"?

### Advanced
- How would you make the case, in a design discussion, for starting simpler and evolving toward complexity only when a specific requirement demands it — against a colleague advocating for adopting a sophisticated pattern preemptively?
- Describe a realistic scenario where starting with a modular monolith and evolving incrementally (introducing CQRS for one read path, splitting off one microservice) worked better than adopting the full sophisticated architecture from day one.

### Follow-up Questions
- Does "avoid unnecessary architecture" mean architecture decisions should never be made proactively?
- Is it always cheaper to add a pattern later than to have started with it from day one?

### Code Prediction
A team adopts full CQRS with separate read/write databases, event-driven synchronization, and a message broker for a small internal admin tool with a handful of users and no read/write performance divergence at all. What's the most likely practical consequence for this team's day-to-day development velocity, compared to a single shared model on one database?

## Practical Tasks

- Apply the four-question checklist to a proposed architectural decision (real or hypothetical) and reach a justified conclusion.
- Identify a pattern from earlier in this module that would be over-engineering for a specific small, simple hypothetical system, and articulate why.
- Design an evolution path for a simple system that would introduce one specific pattern from this module only once a stated, concrete requirement actually appears.

## Readiness Criteria

Apply a consistent, requirement-driven checklist before adopting any architectural pattern, recognize when sophistication is being justified by hypothetical rather than current needs, and design evolution paths that defer complexity until it's genuinely warranted.

## References

### Other

- [Pattern overuse and overengineering (Module 4)](../m04-oop-design/pattern-overuse-and-overengineering.md)
- [Martin Fowler: MonolithFirst](https://martinfowler.com/bliki/MonolithFirst.html)
