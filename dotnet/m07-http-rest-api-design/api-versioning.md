# API Versioning

## Definition

API versioning is the practice of evolving an API's contract over time while keeping existing clients working. A change is a **breaking change** if it could cause an existing, unmodified client to fail or behave incorrectly (removing a field, renaming a field, changing a field's type, changing required-ness); anything else is generally non-breaking and can ship without a version bump.

```
GET /v1/orders/42      — URL path versioning
GET /orders/42
Accept: application/vnd.example.v2+json   — media-type (header) versioning
```

## Alternatives & Trade-offs

**URL path versioning** (`/v1/...`) is the most visible and simplest for clients and tooling (easy to route, easy to see in logs), but "leaks" versioning into every resource's identity/URL. **Header-based versioning** (custom header or `Accept` media type) keeps URLs stable and arguably more RESTful (a resource's identity shouldn't change because the representation format evolved), at the cost of being less visible and slightly harder to test with a browser or simple `curl`. **Query-parameter versioning** (`?api-version=2`) is a middle ground, easy to add without restructuring routes, but easy for clients to omit accidentally and fall back to an unintended default.

## How It Works

### What counts as breaking vs. non-breaking

```
Non-breaking (safe without a version bump):
- Adding a new optional field to a response
- Adding a new endpoint
- Adding a new optional query parameter

Breaking (requires a new version or careful migration):
- Removing or renaming a field
- Changing a field's type (string -> number)
- Making an optional field required
- Changing the meaning of an existing field or status code
```

### URL path versioning

```
GET /v1/orders/42   -> old shape
GET /v2/orders/42   -> new shape, can differ freely from v1
```

Simple to route (often at the reverse-proxy or routing layer) and simple for consumers to pin to a specific version, but means maintaining and testing multiple full route trees over time.

### Header-based versioning

```
GET /orders/42
Accept: application/vnd.example.v2+json
```

Keeps a single canonical URL per resource; the server picks which representation to serve based on the header. This is the more "RESTful" option in the sense that a resource's identity (its URL) doesn't change just because its representation evolves.

### Deprecation strategy

```
GET /v1/orders/42

HTTP/1.1 200 OK
Sunset: Sat, 31 Dec 2026 00:00:00 GMT
Deprecation: true
Link: <https://api.example.com/v2/orders/42>; rel="successor-version"
```

A responsible versioning strategy includes a deprecation window with clear signals (headers, changelog, monitoring which clients still call the old version) before a version is actually removed — removing a version without warning is itself a breaking change to the versioning contract.

## Application

Version an API from the point a breaking change is genuinely needed — don't pre-emptively version `v1` everything before any breaking change has occurred if it adds no value yet, but do establish a versioning *strategy* (which mechanism, what counts as breaking) before the first breaking change forces a rushed decision. Prefer additive, non-breaking changes wherever possible to avoid needing a new version at all.

## Common Mistakes

- Treating "adding a field" as a breaking change and unnecessarily bumping the version for it, fragmenting the API into more versions than needed.
- Bumping a version but not actually maintaining or testing the old version afterward, effectively breaking clients who haven't migrated yet while claiming to still support them.
- Removing an old API version without a deprecation window or clear communication, breaking clients with no warning.
- Choosing a versioning scheme late, after several ad hoc breaking changes have already shipped without any version indicator at all.

## Common Interview Questions

### Basic
- What makes an API change "breaking" versus "non-breaking"?
- What are the main approaches to API versioning?

### Intermediate
- What are the trade-offs between URL-based and header-based versioning?
- Why might adding a new required field to a request be considered breaking even if it seems minor?

### Advanced
- How would you design a deprecation and sunset strategy for an old API version, including how to detect which clients are still using it?
- How does versioning interact with OpenAPI contract generation — should each version have its own document?

### Follow-up Questions
- Is adding a new optional field to a response ever breaking in practice, despite being "non-breaking" by the general rule?
- Can an API avoid versioning entirely by only ever making additive changes?

### Code Prediction
A client's deserializer uses strict mode and fails on unrecognized fields. The API adds a new optional field to a response — technically a "non-breaking" change by the general rule. What happens to this specific client, and what does that reveal about the limits of the breaking/non-breaking rule of thumb?

## Practical Tasks

- Classify a list of proposed API changes (add optional field, remove field, rename field, add endpoint, change status code meaning) as breaking or non-breaking.
- Design a URL-based versioning scheme for an API with one confirmed breaking change coming, including a deprecation timeline for the old version.
- Draft the headers (`Sunset`, `Deprecation`, `Link`) a deprecated endpoint should return.

## Readiness Criteria

Correctly classify breaking versus non-breaking changes including edge cases, choose an appropriate versioning mechanism for a given scenario, and design a responsible deprecation strategy rather than treating versioning as a one-time decision.

## References

### Other

- [RFC 8594: The Sunset HTTP Header Field](https://www.rfc-editor.org/rfc/rfc8594)
- [Microsoft REST API Guidelines — Versioning](https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md)
