# Load and Stress Testing

## Definition

**Load testing** verifies a system behaves correctly and within acceptable performance bounds under an expected, realistic level of traffic. **Stress testing** deliberately pushes beyond expected traffic to find the actual breaking point and observe *how* the system fails — gracefully (rejecting excess requests cleanly) or catastrophically (cascading failures, data corruption, crashes).

```
Load test:   simulate the expected peak traffic (e.g., 500 requests/second) and verify latency/error
             rate stay within acceptable bounds.
Stress test: keep increasing load past that point until something breaks, and observe what breaks
             and how — this is where the real value often is.
```

## Alternatives & Trade-offs

Skipping load testing and discovering capacity limits in production is the highest-risk option — real users experience the failure, at the worst possible time (during genuine peak demand). Load testing in a staging/pre-production environment costs setup effort and infrastructure, but finds capacity limits and performance regressions before they affect real users, with the trade-off that a staging environment rarely perfectly replicates production's exact infrastructure, data volume, and network conditions.

## How It Works

### Realistic load testing — matching expected real-world traffic patterns

```
A load test simulating 500 constant requests/second uniformly is less realistic than one modeling
actual traffic shape: a spike at a known peak time, a mix of read-heavy and write-heavy operations
matching real usage proportions, and realistic data volumes rather than a nearly-empty test database.
```

### Stress testing — finding the breaking point deliberately

```
Gradually increase load past the expected peak until:
  - Latency degrades beyond acceptable bounds (a soft limit — the system is still up, just slow)
  - Error rate spikes (requests start failing outright)
  - The system crashes or becomes unresponsive entirely (a hard limit)
```

The valuable finding from stress testing usually isn't just "the number where it breaks" but *how* it breaks — does it degrade gracefully (returning `503`s and recovering once load drops) or does it fail catastrophically (data corruption, cascading failures across dependent services)?

### What a load/stress test often reveals that unit/integration tests can't

```
- Connection pool exhaustion under concurrent load (Module 13's database-performance topic)
- Thread-pool starvation from blocking calls that only manifests under real concurrency (Module 6)
- A cache stampede on a popular key expiring under high traffic (this module's caching-strategies topic)
- A resource leak that's invisible at low volume but compounds catastrophically at scale
```

None of these show up in a single-request functional test — they're specifically failure modes of *concurrent, sustained* load, which is exactly what load/stress testing is designed to surface.

### Tools (awareness level)

Common tools for generating load include k6, JMeter, Azure Load Testing, and `NBomber` (a .NET-native option) — the specific tool matters less than understanding what realistic load modeling and a genuine stress test should reveal.

## Application

Load-test before a known high-traffic event (a product launch, a marketing campaign) and as a regular part of validating capacity assumptions haven't regressed. Stress-test specifically to understand failure behavior and find the actual breaking point, not just to confirm the expected-load case works — the failure-mode observation is often the more valuable output.

## Common Mistakes

- Load testing with unrealistic traffic patterns (uniform load, tiny test datasets) that don't reveal the same bottlenecks real production traffic would.
- Skipping stress testing entirely because the expected-load case passed, missing the chance to observe (and fix) catastrophic failure behavior before it happens in production under an actual unexpected spike.
- Not testing in an environment resembling production closely enough (data volume, network topology, dependent service behavior) to trust the results.
- Treating a single passing load test as permanent validation, without re-running it as the system evolves and traffic patterns change.

## Common Interview Questions

### Basic
- What's the difference between load testing and stress testing?
- Why is finding the breaking point valuable, beyond just confirming expected load works?

### Intermediate
- What failure modes does load/stress testing reveal that unit and integration tests typically can't?
- Why does a load test with unrealistic traffic patterns risk missing real bottlenecks?

### Advanced
- How would you design a stress test specifically to observe whether a system degrades gracefully or catastrophically under overload?
- How would you decide which dependencies to include for real versus substitute with a stub/mock in a load test, given the trade-off between realism and test environment cost/complexity?

### Follow-up Questions
- Should load testing be a one-time activity before launch, or ongoing?
- Does passing a load test in staging guarantee the same behavior in production?

### Code Prediction
A load test at the expected peak traffic (500 req/s) passes cleanly. A stress test gradually increasing load to 1,500 req/s reveals the connection pool becomes exhausted at 900 req/s, causing cascading timeouts across unrelated endpoints sharing the same pool. What does this specific finding suggest about a capacity limit the load test alone never revealed?

## Practical Tasks

- Design a realistic load test scenario for a specific application, matching expected traffic shape and data volume rather than uniform artificial load.
- Run a stress test past expected capacity and document the specific failure mode observed (graceful degradation vs. catastrophic failure).
- Identify a bottleneck (connection pool exhaustion, thread starvation, cache stampede) that only manifests under sustained concurrent load, not in a single-request test.

## Readiness Criteria

Design realistic load tests matching actual traffic patterns, run stress tests specifically to observe failure behavior at the breaking point, and connect findings back to specific, fixable root causes rather than treating "it broke at X req/s" as the whole answer.

## References

### Other

- [k6 documentation](https://k6.io/docs/)
- [NBomber documentation](https://nbomber.com/docs/overview)
