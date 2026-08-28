# Profilers and Diagnostic Tools

## Definition

A profiler observes a running application to reveal where time (CPU profiling) or memory (memory profiling) is actually being spent, at a level of detail beyond what logs or metrics capture. .NET provides both integrated tooling (Visual Studio's Diagnostic Tools, `dotnet-trace`, `dotnet-counters`, `dotnet-dump`) and third-party profilers for deeper investigation.

```bash
dotnet-counters monitor -p <pid>          # live view of GC, thread pool, and custom metric counters
dotnet-trace collect -p <pid>              # captures a CPU trace for later analysis
dotnet-dump collect -p <pid>                # captures a full memory dump for post-mortem analysis
```

## Alternatives & Trade-offs

Lightweight, always-on tools (`dotnet-counters`, application metrics) are cheap to run continuously and good for noticing *that* something is wrong. Heavier tools (a full CPU trace, a memory dump) capture far more detail but are more expensive to collect and analyze, and are typically used for a specific, already-suspected problem rather than continuously — the right tool depends on whether you're detecting a problem exists or diagnosing one you've already detected.

## How It Works

### `dotnet-counters` — a live, lightweight pulse check

```bash
dotnet-counters monitor -p 1234
# Shows live: CPU usage, GC heap size and collection counts, thread pool queue length, custom app metrics
```

Useful as a first, cheap check when something feels wrong — is the thread pool queue growing unboundedly (a sign of the thread-pool-starvation risk from Module 6)? Is GC running unusually often (tying back to this module's allocation topic)?

### `dotnet-trace` — capturing a detailed CPU profile for later analysis

```bash
dotnet-trace collect -p 1234 --duration 00:00:30
# Produces a trace file, viewable in tools like PerfView or the Visual Studio profiler,
# showing exactly which methods consumed CPU time during the captured window
```

This is the tool that actually answers "measure before optimizing" (this module's first topic) concretely — rather than guessing which method is slow, a CPU trace shows the real call-stack-level breakdown.

### `dotnet-dump` — post-mortem memory analysis

```bash
dotnet-dump collect -p 1234
dotnet-dump analyze core_20260101_120000
> dumpheap -stat   # shows object counts and total size by type — reveals a memory leak's culprit type
```

Useful specifically for diagnosing memory leaks or unexpectedly high memory usage, by inspecting exactly what's on the heap and in what quantity at the moment the dump was taken.

### Visual Studio's integrated Diagnostic Tools — for local development investigation

The Diagnostic Tools window (CPU usage, memory usage, events) integrated directly into the debugger is often the fastest path for a local, interactive investigation before reaching for the command-line tools above, which are more suited to production or CI-triggered investigation where attaching a full debugger isn't practical.

## Application

Reach for lightweight, always-available tools (`dotnet-counters`, existing metrics) first to confirm something is actually wrong and get a rough sense of where. Escalate to heavier, detailed tools (`dotnet-trace` for CPU, `dotnet-dump` for memory) once a specific problem is suspected and needs a precise root cause. Use Visual Studio's integrated tools for fast local iteration during development.

## Common Mistakes

- Reaching immediately for a full memory dump or CPU trace for a vague "something feels slow" report, without first using lightweight tools to narrow down where to look.
- Not having diagnostic tooling available or permitted in production, discovering only during an actual incident that there's no way to capture a trace or dump from the running system.
- Capturing a trace/dump but not knowing how to interpret the output, treating tool output as an answer rather than raw data still requiring analysis.
- Profiling only in a local development environment and assuming the findings transfer directly to production behavior under real load and data volume.

## Common Interview Questions

### Basic
- What's the difference between `dotnet-counters`, `dotnet-trace`, and `dotnet-dump`, and when would you use each?
- Why might you start with a lightweight tool before reaching for a full CPU trace or memory dump?

### Intermediate
- How would you use `dotnet-trace` to identify which specific method is consuming the most CPU time in a slow operation?
- How would you use `dotnet-dump` to diagnose a suspected memory leak?

### Advanced
- How would you set up diagnostic tooling access for a production environment, balancing investigative capability against security and performance overhead?
- How would you correlate a `dotnet-counters` observation (e.g., growing thread pool queue) with the deeper diagnosis needed to actually fix the underlying cause?

### Follow-up Questions
- Can these diagnostic tools be used against a running production process without restarting it?
- Does capturing a CPU trace or memory dump have a performance impact on the running application?

### Code Prediction
`dotnet-counters` shows the GC heap size growing steadily over several hours without ever shrinking, even during periods of low traffic. What would this observation suggest as a next diagnostic step, and which tool would you reach for to confirm it?

## Practical Tasks

- Use `dotnet-counters` to observe a running application's GC and thread-pool behavior under a simulated load test.
- Capture and analyze a `dotnet-trace` CPU profile for a deliberately slow operation, identifying the actual bottleneck method.
- Capture and analyze a `dotnet-dump` memory dump for a deliberately introduced memory leak, identifying the leaking type.

## Readiness Criteria

Choose the appropriate diagnostic tool for a given investigation stage (detection vs. detailed diagnosis), and interpret their output to reach an actual root cause rather than just collecting data.

## References

### Microsoft Learn

- [dotnet-counters](https://learn.microsoft.com/dotnet/core/diagnostics/dotnet-counters)
- [dotnet-trace](https://learn.microsoft.com/dotnet/core/diagnostics/dotnet-trace)
- [dotnet-dump](https://learn.microsoft.com/dotnet/core/diagnostics/dotnet-dump)
