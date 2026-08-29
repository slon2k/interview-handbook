# Availability and Resilience (System Level)

## Definition

Availability is the proportion of time a system is able to serve requests correctly, often expressed as a percentage ("three nines" = 99.9% uptime). Resilience is the system's ability to continue functioning — fully or in a degraded form — despite failures of individual components. Module 13 covered specific resilience *patterns* (timeouts, retries, circuit breakers) at the individual-call level; this topic is about designing for availability across a whole system's components and dependencies.

```
99.9% ("three nines")  -> ~8.7 hours of downtime per year
99.99% ("four nines")  -> ~52 minutes of downtime per year
99.999% ("five nines") -> ~5 minutes of downtime per year
```

## Alternatives & Trade-offs

Higher availability targets cost real engineering effort and infrastructure — redundancy, failover automation, careful dependency management — that scales non-linearly (going from 99.9% to 99.99% is disproportionately harder than going from 99% to 99.9%). Most systems don't need five nines; the right target depends on the actual business cost of downtime, and chasing availability far beyond what's actually justified is itself a form of the "unnecessary architecture" this module warns against elsewhere.

## How It Works

### Redundancy — no single point of failure

```
A single database instance is a single point of failure — its failure means total unavailability
for anything depending on it. A primary/replica setup (or a managed database with automatic
failover) means a single instance failing doesn't take down the whole system, at the cost of
additional infrastructure and replication complexity.
```

### Graceful degradation — staying partially available instead of fully down

```csharp
public async Task<ProductPage> GetProductPageAsync(int id)
{
    var product = await _productService.GetAsync(id); // core functionality — must succeed
    List<Review> reviews;
    try { reviews = await _reviewService.GetReviewsAsync(id); } // non-critical — degrade gracefully if it fails
    catch (Exception) { reviews = new List<Review>(); } // page still loads, just without reviews
    return new ProductPage(product, reviews);
}
```

Distinguishing which dependencies are truly critical (their failure should fail the whole request) from which are non-critical (their failure should degrade the experience, not eliminate it) is a deliberate design decision, not something that happens automatically.

### Redundancy across failure domains, not just redundant instances

```
Running three instances of a service on the same physical rack/availability zone provides
redundancy against a single INSTANCE failing, but not against that whole rack/zone failing.
Spreading instances across multiple availability zones/regions protects against a broader
category of failure, at additional cost and complexity.
```

### Availability as a chain — the weakest critical dependency sets the ceiling

```
If Service A (99.9% available) has a hard dependency on Service B (99% available) for every
request, Service A's own achievable availability is capped by Service B's, regardless of how
reliable Service A's own code is — this is why graceful degradation for non-critical
dependencies matters so much: it removes them from this multiplicative chain entirely.
```

## Application

Set an availability target based on actual business cost of downtime, not an arbitrary aspiration. Identify which dependencies are truly critical (whose failure should fail the request) versus non-critical (whose failure should degrade gracefully), and design explicitly for the latter. Add redundancy across meaningful failure domains (not just redundant instances on the same underlying infrastructure) proportional to the actual availability target.

## Common Mistakes

- Chasing an availability target far beyond what the business impact of downtime actually justifies, spending disproportionate effort for little practical benefit.
- Treating every dependency as equally critical, missing the opportunity to degrade gracefully for genuinely non-critical ones.
- Adding redundant instances without considering whether they share a common failure domain (same rack, same zone) that a single larger-scale failure could still take out entirely.
- Not recognizing that a system's overall availability is capped by its most unreliable *critical* dependency, regardless of how reliable the rest of the system is.

## Common Interview Questions

### Basic
- What does "availability" mean, and how is it typically expressed?
- What's the difference between redundancy and graceful degradation as resilience strategies?

### Intermediate
- Why does a system's overall availability get capped by its least reliable critical dependency?
- How would you decide which dependencies of a service should be treated as critical versus non-critical?

### Advanced
- How would you design a system to achieve a specific availability target, given its dependencies' individual availability numbers?
- What's the difference between redundancy within one failure domain versus across multiple failure domains, and when does the distinction actually matter?

### Follow-up Questions
- Is 99.999% availability an appropriate target for every system?
- Does graceful degradation apply only to reads, or can it apply to writes as well?

### Code Prediction
A service depends synchronously on two other services for every request: one at 99.9% availability, one at 99% availability, with no graceful degradation for either. Roughly what's the best-case availability ceiling for this service, assuming independent failures, and which dependency dominates that ceiling?

## Practical Tasks

- Design graceful degradation for a page/endpoint with one critical and one non-critical dependency.
- Calculate the availability ceiling for a service with several hard synchronous dependencies, and identify which dependency should be prioritized for improvement.
- Propose an appropriate availability target for a hypothetical system based on the described business cost of downtime, rather than defaulting to "as high as possible."

## Readiness Criteria

Explain availability targets and their real cost, distinguish critical from non-critical dependencies and design graceful degradation accordingly, and reason about how dependency chains cap overall achievable availability.

## References

### Other

- [Microsoft: Design principles - design for business needs](https://learn.microsoft.com/azure/architecture/guide/design-principles/)
