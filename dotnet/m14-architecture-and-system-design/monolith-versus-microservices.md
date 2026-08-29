# Monolith vs. Microservices, and Microservice Trade-offs

## Definition

A monolith (or modular monolith, an earlier topic) is deployed as one unit. Microservices split a system into multiple independently deployable services, each typically owning its own data, communicating over the network (this module's sync/async topics). The choice is a trade-off between operational simplicity and independent scalability/deployability — not a maturity ladder where microservices are simply "the advanced version."

```
Monolith:      1 deployable unit, 1 database (typically), simple to develop/test/deploy/debug locally
Microservices: N deployable units, N databases (typically), each independently scalable/deployable,
               but with real network calls, data consistency, and operational complexity between them
```

## Alternatives & Trade-offs

A monolith (ideally modular, per the earlier topic) is simpler to develop, test, deploy, and debug — a single codebase, a single deployment, no network calls between internal components, no distributed-systems complexity. Microservices allow independent scaling (only scale the component that actually needs it) and independent deployment (ship one team's changes without redeploying everything), at the cost of network latency and failure between every inter-service call, data consistency challenges (a later topic), and genuinely harder local development/debugging/testing across service boundaries.

## How It Works

### What microservices actually solve — and what they don't automatically solve

```
Microservices genuinely help when: different components need to scale independently and
dramatically differently, different teams need to deploy independently without coordinating
releases, or different components have genuinely different technology/reliability requirements.

Microservices do NOT automatically solve: bad module boundaries (a poorly-decomposed
microservices system just has the SAME tangled coupling, now over a slower, less reliable
network instead of in-process calls) or team/organizational problems unrelated to deployment
independence.
```

### The "distributed monolith" anti-pattern — microservices without the actual benefit

```
Splitting a system into microservices along boundaries that don't correspond to independent
business capabilities (e.g., splitting a single business transaction across three services that
must always be deployed and changed together anyway) gets ALL the network/consistency cost of
microservices with NONE of the independent-scaling/deployment benefit — a "distributed monolith."
```

This is the single most common microservices mistake, and it's exactly why a well-bounded modular monolith (which enforces the *same* internal boundaries without the network cost) is often the better starting point, per the earlier topic.

### The real cost checklist, honestly assessed

```
- Network calls where there used to be in-process calls (latency, failure modes, this module's
  sync/async topics)
- Data consistency across service boundaries (a later topic — no more free ACID transactions
  spanning what used to be one database)
- Operational overhead: N services to deploy, monitor, version, and debug instead of one
- Local development/testing complexity: reproducing a multi-service system locally is genuinely harder
```

### When to actually make the split

```
The strongest signal: a specific, demonstrated need — a component that must scale to 100x the
rest of the system's load, or a team that genuinely needs to deploy independently on its own
cadence without coordinating with others — not a general belief that microservices are "more
scalable" or "more modern" in the abstract.
```

## Application

Default to a (modular) monolith unless a specific, demonstrated scaling or team-independence need justifies the real cost of microservices. When splitting into services, split along genuine business-capability boundaries (the same boundaries a modular monolith would already enforce internally) rather than arbitrary technical layers, to avoid the distributed-monolith trap.

## Common Mistakes

- Adopting microservices because they're perceived as the more modern or scalable default, without a specific, demonstrated need that a monolith couldn't satisfy.
- Splitting a system along boundaries that don't correspond to independently-deployable business capabilities, creating a distributed monolith that pays microservices' costs without their benefits.
- Assuming microservices automatically fix poor module boundaries or team-coordination problems that were never really about deployment architecture in the first place.
- Underestimating the genuine operational, consistency, and local-development cost of a microservices split before committing to it.

## Common Interview Questions

### Basic
- What's the fundamental trade-off between a monolith and microservices?
- What is a "distributed monolith," and why is it worse than either a true monolith or true microservices?

### Intermediate
- What specific, concrete needs would justify splitting a monolith into microservices?
- Why doesn't splitting into microservices automatically fix poor module boundaries?

### Advanced
- How would you evaluate whether a proposed microservices split follows genuine business-capability boundaries or risks becoming a distributed monolith?
- How does a well-structured modular monolith make a later, genuinely-justified microservices split easier than migrating from an undifferentiated monolith?

### Follow-up Questions
- Is a large number of services always a sign of a mature architecture?
- Can a system reasonably mix a monolith for its core and a few genuinely independent microservices for specific components?

### Code Prediction
A system splits `OrderService` and `PaymentService` into separate microservices, but every deployment still requires both to be updated and released together because they share tightly-coupled internal logic and must always agree on a shared data format. What has this split actually achieved, compared to keeping them as two well-bounded modules within one modular monolith?

## Practical Tasks

- Evaluate a hypothetical system for whether a proposed microservices split follows genuine business-capability boundaries or risks becoming a distributed monolith.
- List the specific operational costs (deployment, monitoring, local development) a team should expect after splitting a monolith into three services.
- Design a decision framework (a short checklist) for when a team should consider splitting a well-bounded modular monolith into real microservices.

## Readiness Criteria

Articulate the honest trade-off between monoliths and microservices without treating either as inherently superior, recognize the distributed-monolith anti-pattern, and evaluate a proposed split against genuine business-capability boundaries.

## References

### Other

- [Martin Fowler: MonolithFirst](https://martinfowler.com/bliki/MonolithFirst.html)
- [Microsoft: Microservices architecture style](https://learn.microsoft.com/azure/architecture/guide/architecture-styles/microservices)
