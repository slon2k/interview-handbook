# Database Migrations During Deployment

## Definition

Module 10 covered EF Core migrations mechanically, including why running `Database.Migrate()` automatically at app startup is risky in a scaled-out production deployment. This topic is the delivery-pipeline angle: how a migration should actually be applied as a controlled, ordered step in a deployment, especially when application code and database schema must both change together without an incompatible window where old code meets new schema (or vice versa).

```
Deployment order matters: if the new application code REQUIRES a new column that doesn't
exist yet, deploying the code before the migration causes immediate failures. If the migration
DROPS a column the OLD code still reads, deploying the migration before all old-code instances
have stopped causes the same problem, just in the other direction.
```

## Alternatives & Trade-offs

Applying a migration automatically at application startup (Module 10's territory) is simple but risky for a scaled-out deployment — multiple instances starting concurrently might race to apply the same migration, and there's no separate point to review or roll back the schema change independently of the code deployment. Applying migrations as an explicit, separate pipeline step (generating a script, running it once, then deploying application code) is more control and more steps, but avoids the race condition and gives a clear, independent point to verify the schema change succeeded before code that depends on it starts running.

## How It Works

### The core hazard — schema and code changes must be sequenced correctly

```
Scenario: adding a new required column.
  1. Migrate the database FIRST, but make the new column NULLABLE initially (not required yet)
     -- OLD code (which doesn't know about this column) keeps working fine
  2. Deploy the new application code that reads/writes the new column
  3. ONLY ONCE all instances are running the new code, run a SECOND migration making the
     column required/non-nullable, backfilling any rows that predate step 2
```

This "expand, migrate code, contract" sequencing avoids ever having a window where deployed code and deployed schema are mutually incompatible — a single combined "add a required column and deploy new code simultaneously" step risks exactly that incompatible window during a rolling deployment where old and new code briefly run side by side.

### Backward-compatible migrations — the general principle behind the example above

```
Prefer migrations that are backward-compatible with the code that's ABOUT to be replaced:
  - Adding a nullable column: safe, old code ignores it
  - Adding a new table: safe, old code doesn't know it exists
  - Renaming or removing a column the OLD code still reads: NOT safe during a rolling deployment
  - Making a column non-nullable that OLD code might still leave null: NOT safe
```

### Migrations as a separate, reviewable pipeline step

```yaml
stages:
  - build-and-test        # Module 11
  - generate-migration-script  # Module 10's `dotnet ef migrations script`
  - apply-migration        # a distinct, auditable step — NOT tied to app startup
  - deploy-application      # only proceeds after the migration step succeeds
```

Separating these into distinct pipeline steps means a migration failure stops the deployment before any application code depending on the new schema is rolled out, and gives an explicit audit trail of exactly when each schema change was applied.

### Rollback considerations specific to migrations

```
Rolling back APPLICATION CODE is usually straightforward (redeploy the previous version).
Rolling back a DATABASE MIGRATION that already ran and potentially transformed/deleted data
is often much harder or outright impossible (Module 10's rename-vs-drop-and-add risk, at its
most consequential) — this is a big part of why backward-compatible, staged migrations (the
expand/contract pattern above) matter: they minimize how often a true schema rollback is ever needed.
```

## Application

Sequence schema and code changes using an expand/contract pattern for any change that isn't trivially backward-compatible, so a rolling deployment never has a window where deployed code and deployed schema are mutually incompatible. Apply migrations as a distinct, reviewable pipeline step rather than automatically at application startup. Design migrations to be backward-compatible with the code they're about to replace wherever practical, since rolling back an already-applied, data-transforming migration is often far harder than rolling back application code.

## Common Mistakes

- Combining a non-backward-compatible schema change and its corresponding code change into one deployment step, creating a window during a rolling deployment where old code and new schema (or vice versa) are incompatible.
- Applying migrations automatically at application startup in a scaled-out production deployment, risking concurrent migration attempts from multiple starting instances (Module 10's specific concern, now at the pipeline-design level).
- Assuming a database migration can always be rolled back as easily as application code, when a data-transforming or destructive migration often can't be undone cleanly.
- Not sequencing a required-column addition through nullable-first, backfill, then non-nullable steps, instead trying to do it all in one migration during a rolling deployment.

## Common Interview Questions

### Basic
- Why is applying a database migration automatically at application startup risky in production?
- What's the "expand, migrate code, contract" pattern for schema changes?

### Intermediate
- Why does adding a required (non-nullable) column need to be split into multiple deployment steps during a rolling deployment?
- Why is rolling back a database migration often harder than rolling back application code?

### Advanced
- How would you sequence a deployment that both renames a column and updates the application code that uses it, without any incompatible window during a rolling deployment?
- How would you design a pipeline that applies migrations as a distinct, auditable step separate from application deployment?

### Follow-up Questions
- Is every schema change equally risky during a rolling deployment, or are some genuinely safe to combine with code changes in one step?
- Does the expand/contract pattern apply to migrations that only add new, optional functionality?

### Code Prediction
A rolling deployment updates instances one at a time. A single combined migration both makes a column non-nullable and the corresponding code change ships in the same deployment step. During the rolling window, some instances are still running OLD code that never sets this column. What happens to write operations from those old-code instances once the migration has already run?

## Practical Tasks

- Design an expand/contract migration sequence for adding a new required column during a rolling deployment.
- Design a pipeline with migrations as a distinct step, separate from application code deployment, including a failure path that stops before code deploys.
- Identify a proposed schema change that would be unsafe to combine with its corresponding code change in one rolling-deployment step, and redesign the sequencing.

## Readiness Criteria

Sequence schema and code changes safely for rolling deployments using expand/contract patterns, apply migrations as a distinct pipeline step rather than at startup, and reason about the asymmetry between rolling back code versus rolling back a migration.

## References

### Microsoft Learn

- [Applying migrations](https://learn.microsoft.com/ef/core/managing-schemas/migrations/applying)

### Other

- [Martin Fowler: Evolutionary Database Design](https://martinfowler.com/articles/evodb.html)
