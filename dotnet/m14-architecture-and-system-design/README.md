# Module 14 - Architecture and System Design Fundamentals

**Status:** Complete  
**Priority:** Medium  
**Prerequisites:** [Object-Oriented Design and Maintainable Code](../m04-oop-design/README.md), [Asynchronous Programming and Concurrency](../m06-async-concurrency/README.md), [Entity Framework Core](../m10-entity-framework-core/README.md)

## Scope

For a mid-level role, this module focuses on explaining reasonable designs and defending trade-offs, not designing global-scale distributed systems from scratch. Several topics here are deliberately the architecture-level angle on things already fully covered elsewhere — repositories (Module 4, 10), caching (Module 8, 13), N+1/consistency concerns (Module 9, 10, 13) — and cross-reference rather than repeat that content. The closing topic, avoiding unnecessary architecture, names the discipline running through the whole module: every pattern here earns its cost only when a real, current requirement justifies it.

## Learning Outcomes

By the end of this module, you should be able to:

- Explain layered/hexagonal architecture and dependency direction, and place DTOs, application services, and repositories correctly within that structure.
- Choose between synchronous and asynchronous communication, and design event-driven flows with correct command/event and queue/topic distinctions.
- Reason about caching topology, relational vs. NoSQL fit, statelessness, and scaling at the system level.
- Design for availability using redundancy and graceful degradation, and articulate the honest monolith-vs-microservices trade-off.
- Design sagas with compensating actions and idempotent message handling for distributed, eventually-consistent operations.
- Apply CQRS and DDD vocabulary precisely and only where justified.
- Evaluate any architectural pattern against a current, demonstrated requirement rather than a hypothetical future one.

## Topics

### 1. Structuring an Application

- [Layered architecture, separation of domain/infrastructure, and dependency direction](layered-architecture-and-boundaries.md)
- [Modular monolith](modular-monolith.md)
- [Clean and hexagonal architecture (conceptual level)](clean-and-hexagonal-architecture.md)

### 2. Application-Layer Building Blocks

- [DTOs vs. domain entities](dtos-versus-domain-entities.md)
- [Application services](application-services.md)
- [Repositories and units of work — the architecture-level view](repositories-and-units-of-work-in-architecture.md)

### 3. Communication Patterns

- [Synchronous vs. asynchronous communication (system level)](synchronous-versus-asynchronous-communication.md)
- [Messaging fundamentals and event-driven architecture](messaging-and-event-driven-architecture.md)

### 4. Data and Scaling

- [Caching — the system-design view](caching-in-system-design.md)
- [Relational vs. NoSQL: when each fits](relational-versus-nosql.md)
- [Stateless application design, and horizontal vs. vertical scaling](stateless-design-and-scaling.md)
- [Availability and resilience (system level)](availability-and-resilience.md)

### 5. Distributed Systems Trade-offs

- [Monolith vs. microservices, and microservice trade-offs](monolith-versus-microservices.md)
- [Distributed transactions, eventual consistency, and idempotent message handling](distributed-transactions-and-eventual-consistency.md)

### 6. Vocabulary and Judgment

- [Basic CQRS awareness](cqrs-awareness.md)
- [Basic domain-driven design vocabulary](domain-driven-design-vocabulary.md)
- [Avoiding unnecessary architecture](avoiding-unnecessary-architecture.md)

## Scope Boundaries

- The general repository-pattern design trade-off belongs in [Module 4](../m04-oop-design/repository.md); EF-Core-specific mechanics belong in [Module 10](../m10-entity-framework-core/repository-pattern-in-ef-core.md); this module covers only where the pattern sits architecturally.
- Caching mechanics belong in [Module 8](../m08-aspnet-core-fundamentals/basic-caching.md); caching strategy (invalidation, in-memory vs. distributed) belongs in [Module 13](../m13-performance-diagnostics-observability/caching-strategies.md); this module covers only caching's topological placement in a multi-service system.
- Resilience patterns (timeouts, retries, circuit breakers) at the individual-call level belong in [Module 13](../m13-performance-diagnostics-observability/resilience-timeouts-retries-circuit-breakers.md); this module covers availability at the whole-system level.
- Async/await mechanics belong in [Module 6](../m06-async-concurrency/README.md); this module's sync-vs-async topic is about system-level communication choice, not language-level concurrency.
- HTTP/REST idempotency mechanics belong in [Module 7](../m07-http-rest-api-design/http-methods-and-idempotency.md); this module applies the same idea to message consumption.

## Suggested Learning Sequence

1. Layered/hexagonal architecture, modular monolith, dependency direction.
2. DTOs, application services, repositories at the architecture level.
3. Synchronous vs. asynchronous communication, messaging, and event-driven architecture.
4. Caching topology, relational vs. NoSQL, statelessness/scaling, availability.
5. Monolith vs. microservices, distributed transactions, eventual consistency, idempotent messaging.
6. CQRS, DDD vocabulary, and avoiding unnecessary architecture.

## Practical Deliverables

- Design project/assembly boundaries for a layered or modular-monolith application that structurally enforce dependency direction.
- Design a use case as a port driven by two different adapters (e.g., HTTP and a message consumer).
- Design an event-driven flow with at least two independent consumers, and an idempotent consumer for at-least-once delivery.
- Evaluate a proposed microservices split against the distributed-monolith anti-pattern.
- Design a saga with compensating actions for a multi-step distributed operation.
- Apply the "avoiding unnecessary architecture" checklist to a real or hypothetical architectural decision.

## Interview Coverage

Each topic should include:

- Basic questions for terminology and conceptual familiarity.
- Intermediate questions involving design trade-offs and when a pattern fits.
- Advanced questions involving system-level design reasoning and honest trade-off articulation.
- Follow-up questions that test judgment about when *not* to apply a pattern, not just how to apply it.
- Code-prediction questions grounded in concrete scenarios, since this module is fundamentally about defending design decisions, not reciting pattern names.

## References

### Other

- [Martin Fowler's architecture writing (bliki)](https://martinfowler.com/bliki/)
- [Microsoft: Azure Architecture Center](https://learn.microsoft.com/azure/architecture/)
