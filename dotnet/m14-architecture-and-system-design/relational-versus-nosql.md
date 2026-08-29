# Relational vs. NoSQL: When Each Fits (Awareness)

## Definition

Relational databases (Module 9) enforce a fixed schema and strong consistency via ACID transactions, optimized for structured data with well-defined relationships. NoSQL is an umbrella term for several genuinely different data models — document stores (MongoDB), key-value stores (Redis, DynamoDB), wide-column stores (Cassandra), and graph databases (Neo4j) — each trading some of the relational model's guarantees or structure for a different access pattern's benefit. This is awareness-level reasoning: knowing *when* each fits, not learning a second query language.

```
Relational: strong consistency, fixed schema, rich relational queries (joins, Module 9) — the
            right default for most transactional business data.
Document:   flexible, per-document schema, fast for "fetch one aggregate as a whole" access
            patterns — good when data is naturally read/written as self-contained documents.
Key-value:  extremely fast, simple lookups by key — good for caching, session storage, feature flags.
```

## Alternatives & Trade-offs

Relational databases give you strong consistency and flexible querying via joins, at the cost of requiring a fixed schema and (at very large scale) more effort to horizontally scale writes. NoSQL options generally trade some of that consistency/query flexibility for a specific access pattern's performance or a genuinely variable/evolving data shape — the right choice depends entirely on what the actual access pattern and consistency requirement are, not on which technology is newer or more fashionable.

## How It Works

### Document stores — when the natural unit of data is a whole "document"

```
A product catalog where each product has wildly different attributes (a book has an author
and ISBN; a laptop has a CPU and RAM) is awkward to model relationally (many nullable columns,
or many small joined tables) but maps naturally onto a document store, where each product is
just stored as its own flexible-shaped document.
```

### Key-value stores — when the access pattern is purely "get this value by this key"

```
A session store, a feature-flag lookup, or a cache (Module 8/13) needs none of a relational
database's query flexibility — it's always "give me the value for this exact key, fast."
A key-value store is purpose-built for exactly that access pattern and little else.
```

### When relational is still the right default, despite NoSQL being available

```
Order/payment/inventory data — anything with real relationships that need to be queried
flexibly (Module 9's joins) and where strong consistency actually matters (Module 9's ACID/
isolation-levels content) — is usually still best served by a relational database. NoSQL's
relaxed consistency models exist for a reason, and that reason doesn't disappear just because
NoSQL is available.
```

### A system can reasonably use both, for different data with different needs

```
Order Service: relational database for orders/payments (needs ACID, relational integrity)
             + Redis (key-value) for session/cache data
             + a document store for a flexible, rarely-queried "product specifications" catalog
```

Using "the best fit per data shape and access pattern" rather than one technology for everything is a common, reasonable pattern — this is sometimes called polyglot persistence.

## Application

Default to a relational database for structured, relationally-connected business data needing strong consistency (orders, payments, inventory, anything with real foreign-key relationships). Consider a document store specifically when data is naturally self-contained and variably-shaped. Consider a key-value store specifically for simple, extremely fast key-based lookups (caching, sessions, feature flags). Justify each choice by the actual access pattern and consistency need, not by trend.

## Common Mistakes

- Choosing a NoSQL database because it seems more modern or scalable, without the underlying access pattern or consistency requirement actually calling for it.
- Trying to force genuinely relational data (with real foreign-key relationships needing joins) into a document store, ending up manually reimplementing relational query logic in application code.
- Assuming "NoSQL" is one thing, when document stores, key-value stores, and graph databases are genuinely different data models suited to different problems.
- Not considering that a single system can reasonably use multiple data store types for different data with different needs, defaulting to "pick one technology for everything."

## Common Interview Questions

### Basic
- What are the main categories of NoSQL databases, and what's each generally good for?
- When would you choose a relational database over NoSQL, and vice versa?

### Intermediate
- Why might a product catalog with highly variable attributes per item be a better fit for a document store than a relational schema?
- What is polyglot persistence, and when would it be a reasonable architectural choice?

### Advanced
- How would you decide, for a specific piece of data in a system, which data store category actually fits its access pattern and consistency needs?
- What consistency trade-offs do most NoSQL databases make compared to a relational database's ACID guarantees, and when is that trade-off acceptable?

### Follow-up Questions
- Does choosing a document store mean giving up all consistency guarantees?
- Is it reasonable for one system to use both a relational database and a NoSQL store for different data?

### Code Prediction
A team models `Orders`, `OrderItems`, and `Customers` — data with clear foreign-key relationships requiring joins and strong consistency — in a document store instead of a relational database, because "NoSQL scales better." What kind of application-level workarounds would they likely need to build to compensate for the document store's lack of native join support?

## Practical Tasks

- For a set of hypothetical data types (orders, session tokens, a flexible product catalog, a social graph), assign the most appropriate data store category and justify each choice.
- Design a polyglot-persistence architecture for a system with genuinely different data needs across its components.
- Identify a case where forcing relational data into a document store would require reimplementing join-like logic in application code, and articulate why a relational database would be simpler.

## Readiness Criteria

Distinguish the major NoSQL categories and their fit, choose between relational and NoSQL based on actual access pattern and consistency needs, and reason about polyglot persistence as a legitimate architectural choice.

## References

### Other

- [Microsoft: Choose the right data store](https://learn.microsoft.com/azure/architecture/guide/technology-choices/data-store-overview)
