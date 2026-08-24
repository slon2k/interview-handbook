# Memory Diagnostics and Performance Review

## Definition

Memory diagnostics use measurements to distinguish allocation pressure, retained objects, fragmentation, and resource leaks. Performance review turns those observations into a focused change.

## Alternatives & Trade-offs

Use a profiler, trace, heap snapshot, or benchmark according to the question. A benchmark measures repeatable code; a heap snapshot shows retention; a runtime trace shows behavior over time.

## How It Works

A practical investigation asks:

1. Is memory being allocated too quickly?
2. Are objects surviving longer than expected?
3. Is a cache or subscription retaining objects?
4. Are large buffers fragmenting the heap?
5. Does the proposed fix improve the measured symptom?

Measure a baseline, change one variable, repeat the measurement, and record the workload and environment.

## Application

- Use allocation counters to find high-allocation paths.
- Use heap snapshots to follow roots to retained objects.
- Use benchmarks for isolated boxing, collection, parsing, or pooling comparisons.
- Review disposal and subscription lifetimes during leak investigations.
- Prefer a simpler fix when its measured benefit is sufficient.

## Common Mistakes

- Optimizing before reproducing the symptom.
- Treating a single timing run as evidence.
- Confusing retained memory with allocation rate.
- Changing multiple variables at once.
- Keeping an optimization that makes ownership or correctness unsafe.

## Common Interview Questions

### Basic

- What is the difference between allocation and retention?
- What does a memory profiler show?
- Why establish a baseline?

### Intermediate

- When would you use a benchmark versus a heap snapshot?
- How can you find a root retaining an object?
- What measurements indicate GC pressure?

### Advanced

- How would you distinguish LOH fragmentation from a managed leak?
- How do workload shape and warmup affect benchmarks?
- How can an optimization improve allocation rate but worsen retention?
- How would you validate a pooling change in production?

### Follow-up Questions

- Why can a faster benchmark be misleading?
- What should be recorded with a performance result?
- When should an optimization be reverted?

### Code Prediction

Which change should be measured first?

```csharp
var result = values.Where(value => value > 0).ToList();
```

## Practical Tasks

- Create a baseline benchmark for boxing and collection growth.
- Inspect a heap snapshot for a static cache or event subscription.
- Compare a pooled and non-pooled implementation under a representative workload.
- Write a short performance review that states evidence, change, and residual risk.

## Readiness Criteria

You should be able to select an appropriate diagnostic method, distinguish allocation from retention, measure a baseline, and reject optimizations that do not justify their complexity.

## References

### Microsoft Learn

- [Fundamentals of garbage collection](https://learn.microsoft.com/dotnet/standard/garbage-collection/fundamentals)
- [Performance](https://learn.microsoft.com/dotnet/framework/performance/)
- [BenchmarkDotNet](https://benchmarkdotnet.org/)
