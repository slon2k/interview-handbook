# Modular Monolith

## Definition

A modular monolith is a single deployable application internally organized into well-bounded modules — each with a clear responsibility and a deliberately narrow, explicit interface to other modules — without the operational complexity of splitting into separately-deployed microservices. It's frequently the pragmatic middle ground between "one big tangled codebase" and "a full microservices architecture."

```
OrderModule    (owns Orders, exposes IOrderService, has its own internal data model)
InventoryModule (owns Inventory, exposes IInventoryService, has its own internal data model)
-- both compiled and deployed as ONE application, but internally as decoupled as separate services --
```

## Alternatives & Trade-offs

A single, undifferentiated monolith is simplest to start but tends to accumulate tangled cross-module dependencies over time, since nothing structurally prevents `OrderModule` from reaching directly into `InventoryModule`'s internals. Splitting into microservices enforces module boundaries at the strongest possible level (separate deployments, separate processes, network calls between them) but pays the full operational cost of distributed systems (see the monolith-vs-microservices topic) from day one. A modular monolith gets much of the internal-boundary discipline of microservices — modules can only interact through explicit interfaces — while keeping one simple deployment unit, and can be split into real services later *if* a genuine scaling or team-boundary need arises.

## How It Works

### Enforcing module boundaries within one codebase

```csharp
// OrderModule can only interact with InventoryModule through this narrow, explicit interface —
// never by directly referencing InventoryModule's internal entities or database tables
public interface IInventoryService
{
    Task<bool> ReserveStockAsync(string sku, int quantity);
}
```

```
Project structure enforcing this at compile time:
  OrderModule.csproj        -> references InventoryModule.Contracts.csproj only
  InventoryModule.csproj    -> implements InventoryModule.Contracts.csproj
  InventoryModule.Contracts.csproj -> the ONLY thing OrderModule is allowed to see
```

Structuring the actual project references so `OrderModule` can only see `InventoryModule.Contracts` (not `InventoryModule` itself) makes the boundary a compile-time guarantee, not just a convention that can be silently violated.

### Each module can own its own data, even within one database

```sql
-- Even sharing one physical database, each module's tables are conceptually owned by that module alone —
-- OrderModule's code should never write directly to Inventory's tables
CREATE SCHEMA orders;
CREATE SCHEMA inventory;
```

This mirrors Module 9's schema-as-namespace concept, used here specifically to reinforce module ownership boundaries even when a full separate database per module isn't (yet) justified.

### The migration path to microservices, if it's ever actually needed

```
Because OrderModule already only talks to InventoryModule through IInventoryService,
splitting InventoryModule into its own deployed service later mainly means changing
IInventoryService's implementation from an in-process call to an HTTP/message-based one —
OrderModule's code barely needs to change, because the boundary was already explicit.
```

This is the concrete practical payoff of investing in the boundary discipline early, even while staying a monolith.

## Application

Default to a modular monolith for most new systems, especially without a clearly demonstrated need (team-scaling, independent deployment cadence, wildly different scaling requirements per component) for full microservices. Enforce module boundaries structurally (project references, explicit contracts) rather than relying on developer discipline alone, so the option to split into real services later stays genuinely open.

## Common Mistakes

- Calling a codebase a "modular monolith" while modules still reach directly into each other's internal types/tables, with no actual enforced boundary — this is just a monolith with folder names, not a modular one.
- Jumping straight to microservices for a new system without first establishing whether a well-structured modular monolith would satisfy the actual requirements at much lower operational cost.
- Assuming module boundaries can be maintained by convention/documentation alone, without any compile-time or structural enforcement, and being surprised when they erode over time.
- Sharing a single database with no schema/table ownership discipline, making it easy for one module to accidentally depend on another's internal data structure.

## Common Interview Questions

### Basic
- What is a modular monolith, and how does it differ from an undifferentiated monolith?
- What is the main trade-off a modular monolith makes compared to microservices?

### Intermediate
- How would you enforce module boundaries at compile time rather than relying on developer discipline?
- Why might a modular monolith be a better starting point than microservices for most new systems?

### Advanced
- How would a well-structured modular monolith make a later migration to real microservices easier than migrating from an undifferentiated monolith?
- How would you structure database ownership within a modular monolith sharing a single physical database?

### Follow-up Questions
- Does a modular monolith require multiple databases?
- Is a modular monolith always the right choice, or are there cases where full microservices are justified from the start?

### Code Prediction
Given the `OrderModule`/`InventoryModule.Contracts` project reference structure above, what would a developer need to do to bypass the intended module boundary and directly access `InventoryModule`'s internal entity type? Does the project structure make this impossible, or just inconvenient?

## Practical Tasks

- Design project/assembly boundaries for a modular monolith with at least two modules communicating only through explicit contracts.
- Identify a place in a hypothetical undifferentiated monolith where one area reaches directly into another's internals, and refactor it behind an explicit interface.
- Sketch the migration path from a well-bounded module in a modular monolith to a separately deployed microservice.

## Readiness Criteria

Explain the modular monolith as a deliberate middle ground, enforce module boundaries structurally rather than by convention, and reason about when it's sufficient versus when full microservices are actually justified.

## References

### Other

- [Simon Brown / Martin Fowler: on modular monoliths (general architectural reference)](https://martinfowler.com/bliki/MonolithFirst.html)
