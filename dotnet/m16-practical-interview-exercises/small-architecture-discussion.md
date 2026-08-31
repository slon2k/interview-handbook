# Small Architecture Discussion

## What This Assesses

A more open-ended, discussion-style exercise: given a described system and a specific pain point or growth scenario, can you reason through trade-offs out loud, ask clarifying questions, and defend a recommendation — drawing on Module 14's whole toolkit, applied with judgment rather than reciting pattern names.

## Format and Time Expectations

Usually 15-20 minutes of back-and-forth conversation rather than a single answer — expect the interviewer to push back on your first proposal ("what if traffic doubled again?", "what if that dependency goes down?") to see how you adapt.

## Exercise 1: A Monolith Under Growing Load

**Problem:** "Our order-processing monolith is struggling under load, specifically the checkout flow. What would you consider?"

**What a strong answer demonstrates:** Asking clarifying questions first (is the bottleneck CPU-bound or I/O-bound? Module 13 — which specific part of checkout is slow, per "measure before optimizing"?) before proposing a solution. A staged response: first check for the cheap fixes (a missing index, N+1, caching a hot read path — Modules 9/10/13) before reaching for anything architecturally bigger. If genuinely warranted by sustained, demonstrated load specific to checkout, discussing splitting *just* checkout into its own scalable service (Module 14) — not a full microservices rewrite — and explaining why that's proportionate.

**Common mistakes:** Jumping straight to "we should move to microservices" without first asking whether the problem is actually a missing index or an N+1 query that would be far cheaper to fix.

## Exercise 2: A Service With a Flaky Dependency

**Problem:** "Our order service calls a third-party payment provider that has occasional outages. Customers are seeing errors during those windows. What would you do?"

**What a strong answer demonstrates:** Proposing resilience patterns proportional to the problem (Module 13) — timeouts, retries with backoff for transient failures, a circuit breaker to fail fast during a sustained outage rather than piling up slow, doomed requests. Discussing whether the operation is idempotent (Module 7) before recommending retries, since a retry on a non-idempotent charge risks a double charge. If asked "what if the outage is really long," discussing graceful degradation (Module 14's availability content) — can checkout proceed with payment deferred/queued, rather than failing outright?

**Common mistakes:** Recommending "just retry more aggressively" without checking whether the underlying operation is safe to retry at all.

## Exercise 3: Splitting Off a Reporting Feature

**Problem:** "Our reporting dashboard's queries are slow and are affecting the main application's database performance. What would you consider?"

**What a strong answer demonstrates:** Recognizing this as exactly the scenario the simple form of CQRS (Module 14) addresses — separate, denormalized read models optimized for the dashboard's actual query patterns, rather than forcing the reporting queries through the same normalized, write-optimized schema. Discussing the eventual-consistency trade-off this introduces (Module 14) and whether the dashboard's use case can tolerate it (usually yes, for a reporting dashboard). If pushed further ("what if the dashboard needs true real-time data"), discussing the actual cost of that requirement versus a slightly-stale-but-much-faster alternative.

**Common mistakes:** Proposing a full separate microservice with its own database and message-based synchronization for what could be solved more simply with a read-replica or a denormalized reporting table within the same database.

## Readiness Criteria

Ask clarifying questions before proposing a solution, check for cheaper fixes before reaching for architecturally bigger ones, explicitly discuss the trade-off of any proposal (not just its benefit), and adapt your recommendation when the interviewer changes the scenario's constraints mid-discussion.

## References

- [Avoiding unnecessary architecture (Module 14)](../m14-architecture-and-system-design/avoiding-unnecessary-architecture.md)
- [Measuring before optimizing (Module 13)](../m13-performance-diagnostics-observability/measure-before-optimizing.md)
- [Basic resilience: timeouts, retries, and circuit breakers (Module 13)](../m13-performance-diagnostics-observability/resilience-timeouts-retries-circuit-breakers.md)
- [Basic CQRS awareness (Module 14)](../m14-architecture-and-system-design/cqrs-awareness.md)
- [Availability and resilience (Module 14)](../m14-architecture-and-system-design/availability-and-resilience.md)
