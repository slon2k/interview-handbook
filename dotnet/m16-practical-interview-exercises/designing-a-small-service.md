# Designing a Small Service

## What This Assesses

A step up from designing a single endpoint: given a small business domain, can you design a coherent service — its data model, layering, and key design decisions — drawing on Modules 4, 9, 10, and 14 together, and defend your choices when pushed on alternatives.

## Format and Time Expectations

A short domain description ("design a service for X"), typically 15-25 minutes, often on a whiteboard or shared doc. Expect follow-up questions probing specific decisions ("why not a NoSQL store here?", "how would this scale?").

## Exercise 1: A URL Shortener

**Problem:** Design a service that shortens long URLs and redirects short codes back to the original URL.

**What a strong answer demonstrates:**
- Data model: a simple table/entity (`ShortCode`, `OriginalUrl`, `CreatedAt`, maybe `ExpiresAt`) — recognizing this domain doesn't need much complexity (Module 14's avoiding-unnecessary-architecture theme).
- Key design decision: how is the short code generated — a counter-based encoding (Base62 of an auto-incrementing ID) vs. a random string with a collision check? Being able to discuss the trade-off (predictable/guessable vs. needing collision handling) is the actual signal here.
- Read-heavy access pattern: redirects vastly outnumber creations — a strong answer proactively mentions caching the short-code-to-URL mapping (Module 8/13/14) given this access pattern.
- Relational vs. NoSQL (Module 14): a strong answer can justify either a simple relational table or a key-value store, given the simple key-based lookup access pattern — key-value fits nicely here, unlike more relationally-connected domains.

**Common mistakes:** Jumping straight to a microservices/event-driven design (Module 14's distributed-monolith warning) for a domain this simple, or not addressing the read-heavy caching opportunity at all.

## Exercise 2: A Simple Inventory System

**Problem:** Design a service tracking product stock levels, supporting reservation during checkout and release if a checkout doesn't complete.

**What a strong answer demonstrates:** Recognizing the concurrency risk (Module 6/9) — two simultaneous checkouts for the last unit of stock — and proposing a concrete mitigation (optimistic concurrency with a `RowVersion`, Module 10, or a database-level constraint preventing negative stock); a reservation *expiry* mechanism (a background job, Module 8, releasing stock reserved but never confirmed within some window); the aggregate boundary (Module 14's DDD content) — is `Stock` part of the `Product` aggregate, or its own?

**Common mistakes:** Not addressing the concurrent-reservation race condition at all, or proposing a fix (like a global lock around all stock changes) that would badly hurt throughput without considering the trade-off explicitly.

## Exercise 3: A Notification Service

**Problem:** Design a service that sends notifications (email, SMS, push) triggered by events elsewhere in the system.

**What a strong answer demonstrates:** Recognizing this as a natural fit for event-driven architecture (Module 14) — other services publish events ("OrderShipped"), this service subscribes and decides which channel(s) to notify through, without the publishers needing to know notification logic exists at all; idempotent message handling (Module 14) so a redelivered event doesn't send a duplicate notification; a Strategy-pattern-style channel abstraction (Module 4) letting new channels be added without modifying existing dispatch logic.

**Common mistakes:** Designing this as a synchronous call from every other service ("call NotificationService.SendEmail() directly") instead of recognizing the natural fit for asynchronous, decoupled event consumption.

## Readiness Criteria

Propose a data model and key design decisions proportional to the domain's actual complexity, proactively identify the specific risk (concurrency, scale, coupling) most relevant to the given domain, and defend a specific choice when asked about an alternative rather than listing options with no stated preference.

## References

- [Avoiding unnecessary architecture (Module 14)](../m14-architecture-and-system-design/avoiding-unnecessary-architecture.md)
- [Optimistic concurrency (Module 10)](../m10-entity-framework-core/optimistic-concurrency.md)
- [Relational vs. NoSQL (Module 14)](../m14-architecture-and-system-design/relational-versus-nosql.md)
- [Messaging fundamentals and event-driven architecture (Module 14)](../m14-architecture-and-system-design/messaging-and-event-driven-architecture.md)
- [Basic domain-driven design vocabulary (Module 14)](../m14-architecture-and-system-design/domain-driven-design-vocabulary.md)
