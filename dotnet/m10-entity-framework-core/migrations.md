# Code-First Migrations

## Definition

Migrations let EF Core generate and apply incremental database schema changes derived from changes to your C# entity model, instead of hand-writing schema-change SQL. Each migration is a generated C# class describing how to move the schema forward (`Up`) and, ideally, back (`Down`).

```bash
dotnet ef migrations add AddOrderStatus
dotnet ef database update
```

## Alternatives & Trade-offs

Code-first migrations keep schema evolution in source control, versioned alongside the code that depends on it, and let most schema changes be applied without hand-writing SQL. The alternative — hand-written SQL migration scripts, or a database-first approach where the schema is designed directly in the database and the model is scaffolded from it — gives more precise control over exactly what SQL runs, which matters for complex changes (data migrations, non-trivial index strategies) that the auto-generated migration might not express optimally.

## How It Works

### Generating and applying a migration

```bash
dotnet ef migrations add AddOrderStatus   # compares current model to the last migration, generates the diff
dotnet ef database update                  # applies any pending migrations to the target database
```

```csharp
public partial class AddOrderStatus : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder) =>
        migrationBuilder.AddColumn<string>(name: "Status", table: "Orders", nullable: false, defaultValue: "Pending");

    protected override void Down(MigrationBuilder migrationBuilder) =>
        migrationBuilder.DropColumn(name: "Status", table: "Orders");
}
```

### Reviewing generated migrations before applying them

A generated migration should always be reviewed, not blindly trusted — EF Core can't always infer the intended behavior for ambiguous changes (renaming a column looks identical to dropping one column and adding another, unless you tell it otherwise).

```csharp
// EF Core might generate this for what was actually a rename:
migrationBuilder.DropColumn(name: "CustomerName", table: "Orders");
migrationBuilder.AddColumn<string>(name: "Name", table: "Orders");
// This DESTROYS existing data. The correct migration for a rename:
migrationBuilder.RenameColumn(name: "CustomerName", newName: "Name", table: "Orders");
```

### Applying migrations at deploy time, not app startup, in production

```csharp
// Common in development, risky in production: runs migrations automatically whenever the app starts
app.Services.GetRequiredService<AppDbContext>().Database.Migrate();
```

```bash
# Preferred for production: generate a SQL script and run it as a controlled deployment step
dotnet ef migrations script --idempotent -o migrate.sql
```

Running `Database.Migrate()` automatically at app startup means every scaled-out instance starting concurrently could attempt to apply the same migration simultaneously, and a bad migration can't be reviewed or rolled back independently of a code deployment.

### Data migrations alongside schema migrations

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.AddColumn<string>(name: "Status", table: "Orders", nullable: true);
    migrationBuilder.Sql("UPDATE Orders SET Status = 'Pending' WHERE Status IS NULL"); // backfill existing rows
    migrationBuilder.AlterColumn<string>(name: "Status", table: "Orders", nullable: false);
}
```

## Application

Review every generated migration before applying it, especially for renames and any change that could be ambiguous. Apply migrations as a controlled deployment step (a generated SQL script, or a dedicated migration job run once) rather than automatically at application startup in production, particularly for scaled-out deployments.

## Common Mistakes

- Blindly trusting a generated migration for a column rename, which EF Core may express as drop-and-add, silently destroying data.
- Running `Database.Migrate()` automatically at startup in a production app scaled to multiple instances, risking concurrent migration attempts.
- Forgetting to write a data migration (backfilling or transforming existing rows) alongside a schema change that requires one, leaving existing data inconsistent with the new schema's assumptions.
- Not keeping migrations in source control alongside the code, losing the ability to reconstruct exactly which schema version corresponds to which code version.

## Common Interview Questions

### Basic
- What does `dotnet ef migrations add` do?
- What's the difference between `Up` and `Down` in a migration?

### Intermediate
- Why should a generated migration for a column rename always be reviewed manually?
- Why is running `Database.Migrate()` at app startup risky in a scaled-out production deployment?

### Advanced
- How would you safely deploy a migration that both changes a column's nullability and needs to backfill existing data?
- How would you generate a reviewable SQL script from pending migrations for a controlled production deployment?

### Follow-up Questions
- Can migrations be rolled back after being applied to a production database?
- Does EF Core detect a column rename automatically, or does it need to be told explicitly?

### Code Prediction
A developer renames a `CustomerName` property to `Name` on the `Order` entity and runs `dotnet ef migrations add RenameCustomerName` without reviewing the output. What does the generated migration likely do to existing `CustomerName` data, and how would using `RenameColumn` explicitly change that outcome?

## Practical Tasks

- Generate a migration for a property rename, inspect the default generated code, and fix it to use `RenameColumn` instead of drop-and-add.
- Write a migration that both changes a column's type and backfills/transforms existing data safely.
- Generate an idempotent SQL script from pending migrations suitable for a controlled production deployment.

## Readiness Criteria

Review generated migrations critically (especially renames), write correct data migrations alongside schema changes, and choose a safe production migration-deployment strategy.

## References

### Microsoft Learn

- [Migrations overview](https://learn.microsoft.com/ef/core/managing-schemas/migrations/)
- [Applying migrations](https://learn.microsoft.com/ef/core/managing-schemas/migrations/applying)
