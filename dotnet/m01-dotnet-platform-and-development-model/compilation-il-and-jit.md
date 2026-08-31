# Compilation to IL, JIT Compilation, and Basic AOT Awareness

## Definition

C# source code compiles first into **IL** (Intermediate Language) — a CPU-independent bytecode, not native machine code — packaged into an assembly. At runtime, the CLR's **JIT** (Just-In-Time) compiler translates IL into native machine code for the actual CPU it's running on, typically method-by-method, the first time each method is called. **AOT** (Ahead-Of-Time) compilation is an alternative that produces native machine code before runtime at all, skipping JIT entirely.

```
C# source (.cs) -> [Roslyn compiler] -> IL (in a .dll/.exe assembly) -> [JIT, at runtime] -> native machine code
```

## Alternatives & Trade-offs

JIT compilation happens at runtime, which means the same compiled assembly (IL) can run unmodified on different CPU architectures — the JIT produces the right native code for whatever machine it's actually running on. This flexibility costs a small amount of startup time (methods are compiled just before their first use) and some memory for the JIT compiler itself. AOT compilation produces native code ahead of time, eliminating that startup JIT cost and often reducing memory footprint, at the cost of losing some of JIT's runtime flexibility and being tied to the specific target architecture the AOT compilation was done for.

## How It Works

### Why IL exists as an intermediate step at all

```
IL is CPU-independent — the SAME compiled assembly can run on x64, ARM64, or any other
architecture the .NET runtime supports, because the JIT (not the original C# compiler)
produces the actual CPU-specific native code, at the point it's actually needed.
```

This is part of what makes a single published, framework-dependent .NET application portable across different machine architectures without recompilation — the portability lives in IL, not in the original source.

### JIT compilation, method by method, on first use

```
The first time SomeMethod() is called, the JIT compiles ITS IL into native code and caches
that native code for the rest of the process's lifetime — subsequent calls to the SAME method
reuse the already-JIT-compiled native code, without recompiling it again.
```

This explains a real, observable phenomenon: an application's very first few requests are often slightly slower than later ones ("JIT warmup"), since methods are being compiled on their first actual use rather than all at once at startup.

### Tiered compilation — a refinement of plain JIT

```
Tier 0: a quick, less-optimized JIT compilation, to get code running fast with minimal
         startup delay.
Tier 1: for methods called frequently ("hot" methods), the JIT later recompiles them with
         heavier optimization, since the extra optimization time pays for itself for code
         that runs many times.
```

Modern .NET does this automatically — a further refinement of the basic JIT model, optimizing for both fast startup and eventual peak throughput.

### AOT — trading runtime flexibility for startup speed

```
Native AOT (modern .NET) compiles the ENTIRE application to native code ahead of time —
no JIT step at runtime at all, faster startup, often smaller memory footprint — at the cost
of being compiled for one specific target architecture, and some runtime features (certain
kinds of reflection-heavy code) working differently or not at all under AOT.
```

AOT is particularly relevant for scenarios where startup time matters a lot (serverless functions, command-line tools invoked frequently) compared to a long-running web server where JIT's startup cost is paid once and amortized over a long process lifetime.

## Application

Understand IL and JIT as the default, flexible execution model for most .NET applications — nothing to configure or think about for ordinary development. Consider Native AOT specifically for startup-latency-sensitive scenarios (serverless, CLI tools) where the trade-offs (single-target-architecture builds, some reflection limitations) are acceptable.

## Common Mistakes

- Assuming compiled .NET assemblies contain native machine code directly, rather than CPU-independent IL that's translated at runtime.
- Not accounting for JIT warmup when interpreting "the first request was slower" observations during load testing or production monitoring.
- Assuming AOT is a strictly better default with no trade-offs, missing its architecture-specific and reflection-related limitations.
- Confusing tiered compilation's Tier 0/Tier 1 distinction with JIT vs. AOT — tiered compilation is a refinement within the JIT model, not an alternative to it.

## Common Interview Questions

### Basic
- What is IL, and why does C# compile to it instead of directly to native code?
- What does the JIT compiler do, and when does it run?

### Intermediate
- Why might an application's first few requests be slightly slower than later ones?
- What is Native AOT, and what does it trade away compared to standard JIT compilation?

### Advanced
- How does tiered compilation balance fast startup against eventual peak performance?
- In what scenarios would Native AOT's trade-offs be clearly worth it, versus scenarios where standard JIT is the better default?

### Follow-up Questions
- Does IL run at the same speed as native code, or does it need to be translated first?
- Can the same compiled assembly (IL) run on different CPU architectures without modification?

### Code Prediction
A serverless function using standard JIT compilation is invoked infrequently, with each invocation starting a fresh process. Given JIT's per-method, on-first-use compilation model, what specific cost does this workload pay repeatedly that a long-running web server mostly avoids?

## Practical Tasks

- Explain, for a given application type (a long-running web API vs. a frequently-invoked serverless function), whether Native AOT's trade-offs would likely be worth adopting.
- Describe, in your own words, the full path from C# source code to an executing method call, naming each transformation step.

## Readiness Criteria

Explain the IL/JIT model precisely, including tiered compilation, and reason about when Native AOT's trade-offs are appropriate for a given workload.

## References

### Microsoft Learn

- [Managed execution process](https://learn.microsoft.com/dotnet/standard/managed-execution-process)
- [Native AOT deployment overview](https://learn.microsoft.com/dotnet/core/deploying/native-aot/)
