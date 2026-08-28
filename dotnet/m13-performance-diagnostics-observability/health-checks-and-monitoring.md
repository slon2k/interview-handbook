# Health Checks in a Monitoring and Observability Context

## Definition

Module 8 covered implementing health checks (liveness vs. readiness, `Degraded` vs. `Unhealthy`); this topic covers health checks from the *consuming* side — how orchestrators, load balancers, and monitoring dashboards actually use that signal, and how health-check data fits into a broader observability picture alongside metrics, traces, and logs.

```
A health check answers ONE narrow question at a point in time: "can this instance serve traffic right now?"
Metrics/traces/logs answer broader questions: "how well is the system performing, and why?"
```

## Alternatives & Trade-offs

A health check is cheap, fast, and gives a clear binary(ish) signal well-suited to automated infrastructure decisions (route traffic here or don't; restart this instance or don't). It's not designed to diagnose *why* something is unhealthy or to reveal gradual degradation trends — that's what metrics and tracing are for. Treating health checks as your only observability signal misses the aggregate trend and root-cause detail a real incident investigation needs.

## How It Works

### Health checks feeding automated infrastructure decisions

```yaml
# Kubernetes readiness probe — automatically removes this pod from load-balancer rotation if it fails
readinessProbe:
  httpGet: { path: /health/ready, port: 8080 }
  periodSeconds: 10
  failureThreshold: 3
```

The orchestrator acts on this signal automatically, without a human in the loop — which is exactly why the liveness/readiness distinction from Module 8 matters so much: an automated system will restart or reroute based on this signal, so a poorly-designed check can cause unnecessary churn.

### Health-check history as a monitoring dashboard signal, not just a live gate

```
A dashboard tracking "percentage of instances reporting Healthy over the last 24 hours" reveals
a pattern (e.g., health degrades every day at 3 AM during a batch job) that a single live health
check, checked only at the moment of a request, would never surface.
```

Aggregating health-check results over time turns a purely reactive signal into a trend-detection tool, closer to what a metric provides.

### Correlating a health-check failure with the rest of the observability stack

```
Health check reports "Unhealthy: Redis unreachable" at 14:32.
Metrics show error rate spiking at 14:31.
Traces from 14:31-14:35 show calls to the cache timing out.
Logs show the specific Redis connection exception.
```

None of these four signals alone tells the complete story — a health check narrows down "what's currently broken" quickly, while metrics/traces/logs (already covered in this module) provide the depth needed to actually understand and fix the underlying cause.

### Health checks as part of a deployment pipeline, not just runtime monitoring

```
A canary or blue-green deployment can gate promotion of a new version on its health checks
passing consistently for some period, before shifting more traffic to it — connecting health
checks to Module 15's deployment-workflow concerns as well as pure runtime monitoring.
```

## Application

Treat health checks as one fast, automated signal feeding infrastructure decisions (routing, restarts, deployment gating) — not as a substitute for metrics, tracing, and logs when actually diagnosing why something is unhealthy. Aggregate health-check results over time for trend visibility, and correlate a health-check failure with the richer observability signals to find root cause quickly.

## Common Mistakes

- Treating health-check status as sufficient observability on its own, without metrics/tracing/logs to explain *why* a check is failing.
- Not aggregating health-check history over time, missing recurring patterns that only show up when viewed as a trend rather than a single live check.
- Designing a health check so sensitive that transient issues trigger unnecessary automated restarts or traffic rerouting, causing more disruption than the underlying issue would have on its own (the same over-sensitive-liveness-check risk from Module 8, viewed from the monitoring side).
- Not connecting health-check failures to the rest of the observability stack during an incident, missing the faster root-cause path that correlation would provide.

## Common Interview Questions

### Basic
- What decisions do orchestrators and load balancers typically make based on health-check results?
- Why isn't a health check alone sufficient for diagnosing a performance or reliability problem?

### Intermediate
- How would aggregating health-check results over time reveal something a single live check wouldn't?
- How would you correlate a health-check failure with metrics, traces, and logs to find root cause quickly?

### Advanced
- How would you design a deployment pipeline that gates promotion of a new version on sustained health-check success, rather than a single passing check?
- What's the risk of an overly sensitive health check from a monitoring/automation perspective, beyond just the direct restart-churn cost covered in Module 8?

### Follow-up Questions
- Should health-check history be retained and analyzed the same way metrics are?
- Can a health check ever replace the need for tracing during an incident investigation?

### Code Prediction
A health check reports `Unhealthy` for a specific instance three times in a row, and the orchestrator restarts it. If the underlying cause is a slow-draining connection pool rather than a genuine crash, what happens after the restart, and what additional signal (beyond the health check itself) would reveal the real root cause?

## Practical Tasks

- Design an automated deployment gate that requires sustained (not just single) health-check success before promoting a new version.
- Build a simple dashboard aggregating health-check pass/fail history over time for a service, identifying a recurring pattern.
- Correlate a simulated health-check failure with corresponding metrics, trace, and log data to reconstruct the likely root cause.

## Readiness Criteria

Use health checks as one signal among several rather than a complete observability solution, aggregate results for trend visibility, and correlate health-check data with metrics/traces/logs during incident investigation.

## References

### Microsoft Learn

- [Health checks in ASP.NET Core](https://learn.microsoft.com/aspnet/core/host-and-deploy/health-checks)
