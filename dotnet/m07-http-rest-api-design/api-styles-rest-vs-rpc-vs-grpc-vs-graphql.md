# API Styles: REST vs. RPC vs. gRPC vs. GraphQL

## Definition

Four common ways to expose functionality over a network, differing in how they model operations and data: **REST** exposes resources manipulated via standard HTTP methods; **RPC** exposes direct remote procedure calls (verb-oriented, not resource-oriented); **gRPC** is a specific high-performance RPC framework using HTTP/2 and Protocol Buffers; **GraphQL** exposes a single endpoint where clients specify exactly what data shape they want in a query.

```
REST:    GET /orders/42
RPC:     POST /getOrder {"orderId": 42}
gRPC:    rpc GetOrder(GetOrderRequest) returns (Order);   // defined in a .proto file, binary wire format
GraphQL: query { order(id: 42) { id total items { name price } } }
```

## Alternatives & Trade-offs

| | REST | gRPC | GraphQL |
|---|---|---|---|
| Payload format | Usually JSON (text, human-readable) | Protocol Buffers (binary, compact, fast) | Usually JSON |
| Browser-friendly | Yes, natively | No (needs grpc-web or a proxy) | Yes |
| Over-fetching/under-fetching | Common (fixed response shapes) | N/A (defined per-call) | Solved by design (client specifies fields) |
| Streaming | Limited (polling, SSE, WebSockets) | Native bidirectional streaming | Limited (subscriptions, less standard) |
| Best fit | Public APIs, CRUD-heavy services, broad client compatibility | Internal service-to-service calls needing low latency/high throughput | Clients needing flexible, nested data shapes (e.g., mobile apps aggregating several resources) |
| Tooling maturity | Extremely mature, universal | Mature, but less universal client support | Mature but adds a schema/resolver layer |

## How It Works

### REST — resource-oriented, HTTP-native

```
GET /customers/7/orders
```
Simple, cacheable by standard HTTP infrastructure, human-readable — but a mobile client wanting only order totals still receives full order objects unless a separate, narrower endpoint exists (over-fetching).

### gRPC — contract-first, binary, fast

```protobuf
service OrderService {
  rpc GetOrder (GetOrderRequest) returns (Order);
}
message GetOrderRequest { int32 order_id = 1; }
```
The `.proto` file is the single source of truth for the contract, and code is generated for both client and server in whatever languages are needed. Binary serialization and HTTP/2 multiplexing make gRPC significantly faster than JSON-over-HTTP/1.1 for high-volume internal traffic, at the cost of not being directly callable from a browser or easily inspectable with `curl`.

### GraphQL — client-specified shape

```graphql
query {
  order(id: 42) {
    total
    customer { name }
  }
}
```
The client asks for exactly `total` and `customer.name` — nothing more — solving over-fetching by design. The trade-off is server-side complexity: a resolver layer, N+1 query risks across resolvers (analogous to the EF Core N+1 problem, but at the GraphQL field-resolution level), and less straightforward HTTP-level caching since almost everything goes through one endpoint with `POST`.

### Mixing styles in one system

A backend commonly exposes REST or GraphQL externally (broad client compatibility) while using gRPC for internal service-to-service calls (performance) — the styles aren't mutually exclusive across a system's boundaries.

## Application

Default to REST for public-facing and most internal HTTP APIs, given its universal tooling and simplicity. Reach for gRPC specifically for internal, high-throughput, latency-sensitive service-to-service communication where both ends are services you control. Consider GraphQL when clients (especially mobile or multiple heterogeneous frontends) need flexible, nested data shapes and over/under-fetching is a real, measured problem — not by default.

## Common Mistakes

- Choosing gRPC for a public API that needs browser access without accounting for the extra proxy layer (grpc-web) required.
- Adopting GraphQL to solve a problem that a couple of well-designed REST endpoints (or a `?fields=` query parameter) would solve more simply.
- Assuming one style is strictly "better" rather than fit-for-purpose — the right choice depends on the client population, performance needs, and team familiarity.
- Treating this as a purely theoretical comparison in an interview instead of being able to justify a choice for a stated scenario.

## Common Interview Questions

### Basic
- What is the main difference between REST and RPC-style APIs?
- What serialization format does gRPC typically use, and why?

### Intermediate
- What problem does GraphQL solve that REST commonly runs into?
- Why isn't gRPC typically used for public, browser-facing APIs?

### Advanced
- How would you decide between REST, gRPC, and GraphQL for a system with both a public web client and internal microservices?
- What is the N+1 problem in GraphQL resolvers, and how does it parallel the EF Core N+1 problem?

### Follow-up Questions
- Can gRPC and REST coexist in the same system?
- Does GraphQL eliminate the need for API versioning?

### Code Prediction
A mobile app needs a customer's name, their last three orders' totals, and nothing else, from a REST API that returns full customer and order objects. What two API-design approaches (without adopting GraphQL) could reduce this over-fetching?

## Practical Tasks

- For a stated scenario (public API, internal microservices, mobile app aggregating data), justify a choice of REST, gRPC, or GraphQL and explain the trade-offs of the alternatives.
- Sketch a `.proto` service definition for a simple order-lookup gRPC service.
- Identify an over-fetching problem in a REST API design and propose two different fixes (a narrower endpoint, or a query-parameter-based field selector).

## Readiness Criteria

Compare REST, RPC, gRPC, and GraphQL accurately across payload format, performance, and client compatibility, and justify a style choice for a given system's actual constraints rather than by default preference.

## References

### Other

- [gRPC official documentation](https://grpc.io/docs/)
- [GraphQL official documentation](https://graphql.org/learn/)
- [Protocol Buffers documentation](https://protobuf.dev/overview/)
