# Database Query Performance and N+1 Problems

## Definition

This topic is the diagnostic/operational counterpart to Module 9's indexing/execution-plan content and Module 10's N+1 topic: how you actually *notice* a database-driven performance problem in a running system — through query counts, slow-query logs, and execution-plan inspection — rather than spotting it by reading code alone.

```
Symptom: an endpoint that lists 50 orders takes 3 seconds.
Investigation: enable query logging (Module 10) — reveals 51 SQL queries executed for one request.
Root cause: N+1, from lazy-loaded Customer navigation properties (Module 10's n-plus-one-queries.md).
```

## Alternatives & Trade-offs

Spotting N+1 or a missing index by reading code alone is possible but unreliable — the code for a lazy-loading N+1 bug looks completely ordinary, as Module 10 covered. Actually measuring — query counts per request, slow-query logs, execution plans — turns "I suspect this might be slow" into a confirmed diagnosis with a specific, fixable root cause.

## How It Works

### Query-count-per-request as a health metric

```csharp
// EF Core interceptors can count queries per logical operation, surfacing N+1 patterns systematically
// rather than relying on someone noticing it in a specific code review
public class QueryCountInterceptor : DbCommandInterceptor
{
    public static int QueryCount;
    public override InterceptionResult<DbDataReader> ReaderExecuting(DbCommand command, CommandEventData eventData, InterceptionResult<DbDataReader> result)
    {
        Interlocked.Increment(ref QueryCount);
        return result;
    }
}
```

Some teams add a middleware-level assertion in integration tests: "this endpoint should never execute more than N queries" — turning N+1 regressions into an automated test failure instead of a production surprise.

### Slow-query logs — the database's own record of what's actually expensive

```sql
-- SQL Server: Query Store, or extended events capturing queries exceeding a duration threshold
-- PostgreSQL: log_min_duration_statement setting, logging any query slower than the configured threshold
```

A slow-query log operates independently of application code — it catches expensive queries regardless of which code path triggered them, including ones from ad hoc reporting, migrations, or third-party integrations that application-level logging might miss.

### Execution plans in production, not just locally

```
A query might perform well against a local development database with a small dataset,
and only reveal a missing-index problem once production data volume is large enough for
the difference between an index seek and a table scan (Module 9) to actually matter.
```

### Connection pool exhaustion — a database-adjacent bottleneck that isn't about a single slow query

```
Symptom: requests time out waiting to even START a database operation.
Diagnosis: connection pool exhaustion — too many concurrent operations holding connections,
           often caused by connections not being released promptly (e.g., a long-held transaction,
           or DbContext instances not being disposed correctly).
```

## Application

Instrument query counts per logical operation (via interceptors or integration-test assertions) to catch N+1 regressions systematically. Enable and periodically review slow-query logs in production, not just local development. Re-verify execution plans against production-scale data, not just a local development database's small dataset.

## Common Mistakes

- Only checking for N+1 or missing indexes during local development against small datasets, missing problems that only appear at production scale.
- Relying entirely on manual code review to catch N+1 patterns, when an automated query-count assertion in integration tests catches regressions systematically.
- Not enabling or reviewing slow-query logs in production, missing expensive queries triggered by code paths (reporting, migrations, ad hoc scripts) that wouldn't be caught by application-level query logging alone.
- Diagnosing "the database is slow" without distinguishing a genuinely slow query from connection pool exhaustion, which looks similar from the application's perspective but has a completely different fix.

## Common Interview Questions

### Basic
- What tools or techniques would you use to detect an N+1 query problem in a running application?
- What is a slow-query log, and what does it capture that application-level logging might miss?

### Intermediate
- How would you write an automated test that catches an N+1 regression before it reaches production?
- Why might a query perform fine locally but poorly in production?

### Advanced
- How would you diagnose connection pool exhaustion, and how would you distinguish it from a genuinely slow individual query?
- How would you design a systematic, ongoing process for catching database performance regressions, rather than relying on ad hoc investigation after a complaint?

### Follow-up Questions
- Does a slow-query log catch queries from sources other than the main application (migrations, ad hoc scripts)?
- Should query-count assertions be part of a CI test suite?

### Code Prediction
An endpoint's integration test asserts that fetching a list of 10 orders should execute at most 2 SQL queries. A developer introduces a lazy-loaded navigation property access inside the response-building loop. What happens to this test, and how does that compare to relying on manual code review alone to catch the regression?

## Practical Tasks

- Implement a query-count interceptor or equivalent and add an assertion to an integration test catching N+1 regressions.
- Enable slow-query logging for a local database and identify a deliberately introduced expensive query.
- Diagnose a simulated connection-pool-exhaustion scenario and distinguish it from a single slow query.

## Readiness Criteria

Systematically detect N+1 and slow-query problems using measurement rather than code review alone, verify execution plans against production-scale data, and distinguish connection pool exhaustion from individual query slowness.

## References

### Microsoft Learn

- [EF Core interceptors](https://learn.microsoft.com/ef/core/logging-events-diagnostics/interceptors)
- [Related data performance considerations](https://learn.microsoft.com/ef/core/performance/efficient-querying)

### Other

- [PostgreSQL: log_min_duration_statement](https://www.postgresql.org/docs/current/runtime-config-logging.html)
