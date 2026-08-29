# Stateless Application Design, and Horizontal vs. Vertical Scaling

## Definition

A **stateless** application instance holds no client-specific state between requests — anything needed to process a request is either in the request itself or fetched from an external, shared store (a database, a distributed cache). This is what makes **horizontal scaling** (adding more instances) actually work: any instance can handle any request, since none of them holds unique, unshareable state. **Vertical scaling** (making one instance bigger — more CPU/RAM) is the alternative that doesn't require statelessness but has a hard ceiling.

```
Stateful instance:   holds session data in local memory -> only THAT instance can serve THAT user's
                     next request -> "sticky sessions" required, undermining free horizontal scaling.
Stateless instance:  session data lives in a shared store (Redis) -> ANY instance can serve ANY
                     request -> a load balancer can freely distribute traffic across all instances.
```

## Alternatives & Trade-offs

Vertical scaling (a bigger server) is simpler — no architectural changes needed — but has a hard physical/cost ceiling, and a single instance is a single point of failure. Horizontal scaling (more instances) has no such ceiling and provides redundancy, but *requires* statelessness (or explicit externalized state) to work correctly — without it, a load balancer distributing requests across instances would inconsistently see different state depending on which instance happened to handle a given request.

## How It Works

### The stateful trap — why it silently breaks horizontal scaling

```csharp
public class ShoppingCartController : ControllerBase
{
    private static readonly Dictionary<string, Cart> _cartsByUserId = new(); // in-process, per-instance state

    public IActionResult AddItem(string userId, Item item)
    {
        _cartsByUserId[userId].Items.Add(item); // only visible to THIS specific instance
    }
}
```

If a load balancer routes this user's next request to a *different* instance, that instance's `_cartsByUserId` dictionary has no idea this cart exists — the user's cart appears to have randomly emptied, purely as an artifact of which instance happened to handle which request.

### The stateless fix — externalize the state

```csharp
public class ShoppingCartController : ControllerBase
{
    private readonly IDistributedCache _cache; // shared, external store — any instance can read/write it

    public async Task<IActionResult> AddItem(string userId, Item item)
    {
        var cart = await _cache.GetCartAsync(userId); // works identically regardless of WHICH instance handles this
        cart.Items.Add(item);
        await _cache.SaveCartAsync(userId, cart);
    }
}
```

Now any of N instances can handle any request for any user, since the actual state lives outside any individual instance — this is the concrete precondition that makes adding more instances (horizontal scaling) actually work correctly, not just "technically running more copies."

### "Sticky sessions" as a workaround, and why it's a workaround, not a real fix

```
A load balancer configured for "sticky sessions" routes the same user's requests to the same
instance every time, working around the stateful-instance problem above WITHOUT actually making
the application stateless. This avoids the immediate bug, but still means that specific instance
going down loses that user's state entirely, and load can't be redistributed as freely.
```

### When vertical scaling is still the pragmatic choice

```
For a system with a genuine hard ceiling far above current and projected load, or where the
cost/complexity of horizontal scaling and statelessness isn't yet justified, scaling vertically
first (bigger server) is a perfectly reasonable, simpler starting point — this connects directly
to the "avoiding unnecessary architecture" theme running through this module.
```

## Application

Design new services to be stateless by default — any state that must persist across requests belongs in a shared, external store, not in-process memory — specifically so horizontal scaling remains available without redesign later. Reach for vertical scaling as a simpler first step when load doesn't yet justify the complexity of a fully stateless, horizontally-scaled design.

## Common Mistakes

- Storing per-user or per-session state in an in-process static field or `IMemoryCache`, silently breaking correctness the moment the application is scaled to more than one instance.
- Relying on sticky sessions as a permanent fix rather than recognizing it as a workaround that still leaves state vulnerable to a single instance's failure.
- Assuming horizontal scaling "just works" by running more copies of a stateful application, without addressing the underlying state-externalization requirement first.
- Prematurely designing for horizontal scaling and full statelessness for a system whose actual load would be perfectly well served by simpler vertical scaling.

## Common Interview Questions

### Basic
- What does it mean for an application to be stateless, and why does that matter for scaling?
- What's the difference between horizontal and vertical scaling?

### Intermediate
- Why does storing session data in an in-process dictionary break correctness once an application is scaled to multiple instances?
- What is a "sticky session," and why is it a workaround rather than a true fix for statefulness?

### Advanced
- How would you migrate a stateful application (storing session data in-process) to a stateless design supporting horizontal scaling?
- When is vertical scaling still the more pragmatic choice than investing in statelessness and horizontal scaling?

### Follow-up Questions
- Does statelessness mean an application can never have any server-side state at all?
- Can horizontal and vertical scaling be combined?

### Code Prediction
Given the `_cartsByUserId` in-process dictionary example, if the application is scaled from 1 instance to 3 behind a load balancer with no sticky sessions configured, what's the probability a given user's second request in a session is served by the same instance that handled their first, assuming random distribution?

## Practical Tasks

- Refactor an in-process, stateful session-storage implementation into a stateless design using a shared distributed cache.
- Design a scaling strategy for a system, justifying whether vertical scaling alone is currently sufficient or horizontal scaling (and the statelessness it requires) is warranted.
- Identify all the places in a hypothetical application where state might accidentally be held in-process, and externalize each.

## Readiness Criteria

Explain why statelessness is a precondition for effective horizontal scaling, design applications to externalize state appropriately, and judge when vertical scaling remains the simpler, sufficient choice.

## References

### Other

- [Microsoft: Design principles - self-healing (touches statelessness)](https://learn.microsoft.com/azure/architecture/guide/design-principles/self-healing)
