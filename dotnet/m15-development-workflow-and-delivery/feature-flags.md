# Feature Flags

## Definition

A feature flag is a runtime toggle controlling whether a piece of code path is active, letting a team deploy code without immediately activating it, enable a feature for a subset of users, or turn a problematic feature off instantly without a full redeploy. This is the concrete mechanism that makes trunk-based development (Module 15's branching topic) practical for incomplete work — merge frequently, hide behind a flag until ready, rather than keeping a long-lived branch open.

```csharp
if (await _featureManager.IsEnabledAsync("NewCheckoutFlow"))
{
    return await _newCheckoutService.ProcessAsync(order);
}
return await _legacyCheckoutService.ProcessAsync(order);
```

## Alternatives & Trade-offs

Keeping incomplete work on a long-lived branch until it's finished avoids feature-flag code complexity, but reintroduces exactly the merge-conflict and large-diff risk the branching topic described. Feature flags let incomplete work merge to the trunk continuously (hidden behind the flag), at the cost of the flag itself being extra code (a conditional branch) that must eventually be cleaned up — a flag left in the codebase indefinitely after its purpose is served becomes its own form of clutter and confusion.

## How It Works

### Release flags — hiding incomplete work, the trunk-based-development enabler

```csharp
if (await _featureManager.IsEnabledAsync("NewCheckoutFlow")) { /* new, in-progress code */ }
else { /* existing, stable code path */ }
```

This lets `NewCheckoutFlow`'s code merge to the trunk in small increments over time, fully hidden from real users until it's actually ready — solving the exact problem long-lived feature branches create, without needing a long-lived branch at all.

### Operational/kill-switch flags — instant rollback without a redeploy

```csharp
if (await _featureManager.IsEnabledAsync("UseNewPaymentGateway"))
{
    return await _newGateway.ChargeAsync(order); // just found to have a bug in production
}
return await _legacyGateway.ChargeAsync(order); // instantly falls back, flag flip, no redeploy needed
```

If a newly-released feature turns out to have a production problem, flipping its flag off is immediate — far faster than the redeploy-a-previous-version rollback process, and doesn't require reverting any code.

### Percentage rollout and targeted rollout — de-risking a release gradually

```csharp
// Enable for 5% of users first, watching metrics (Module 13) before expanding
var enabled = await _featureManager.IsEnabledAsync("NewCheckoutFlow", new TargetingContext { UserId = userId });
```

Gradually expanding a flag's rollout percentage while watching error rates and other metrics lets a team catch a problem affecting a small fraction of traffic before it affects everyone — a much lower-risk release shape than "on for 100% of users the moment it deploys."

### Flag cleanup — the discipline that prevents permanent clutter

```
Once NewCheckoutFlow is fully rolled out and stable, and the OLD code path is no longer needed
at all, BOTH the flag check AND the old code path should be removed — leaving them in place
indefinitely means every future reader has to understand a conditional branch that no longer
represents a real, live decision.
```

## Application

Use release flags to enable trunk-based development for larger, multi-PR features. Use operational/kill-switch flags for anything risky enough to want an instant, redeploy-free rollback path. Use percentage/targeted rollout to de-risk a release gradually rather than flipping a flag to 100% immediately. Remove flags and their now-dead old code paths once a feature is fully, stably rolled out — treat flag cleanup as part of the feature's actual completion, not an optional follow-up.

## Common Mistakes

- Leaving a feature flag (and its now-unused old code path) in the codebase indefinitely after the feature is fully rolled out and stable, accumulating permanent clutter.
- Using a long-lived branch instead of a feature flag for incomplete work, reintroducing the merge-conflict and large-diff risk feature flags exist to avoid.
- Rolling a risky feature out to 100% of users immediately instead of gradually, missing the chance to catch a problem affecting a small fraction of traffic before it affects everyone.
- Not treating operational flags as a genuine incident-response tool, forgetting they exist and reaching for a full redeploy rollback instead when an instant flag flip would be much faster.

## Common Interview Questions

### Basic
- What is a feature flag, and what problem does it solve for trunk-based development?
- What's the difference between a release flag and an operational/kill-switch flag?

### Intermediate
- How does a feature flag enable an instant rollback without a redeploy?
- Why is flag cleanup an important discipline, not just an optional nicety?

### Advanced
- How would you design a gradual, percentage-based rollout for a risky new feature, including what metrics you'd watch before expanding it further?
- How do feature flags relate to the branching/PR discipline covered elsewhere in this module — what specific problem do they solve that small PRs alone don't?

### Follow-up Questions
- Should every new feature be built behind a flag, or only certain kinds?
- Does removing a feature flag after full rollout also require removing the old, now-unused code path?

### Code Prediction
A newly-released feature behind an operational flag is found to have a bug affecting real transactions in production. The team has two options: flip the flag off, or revert the code and redeploy. Which is faster, and why does that speed difference matter in an active-incident context?

## Practical Tasks

- Implement a release flag hiding an in-progress feature, allowing its code to merge to the trunk before the feature is complete.
- Implement an operational flag for a risky new code path, and practice flipping it off as a simulated incident response.
- Design a gradual, percentage-based rollout plan for a hypothetical risky feature, including the metrics that would gate expanding it further.

## Readiness Criteria

Use release flags to enable trunk-based development, use operational flags for fast, redeploy-free rollback, design gradual rollouts, and treat flag cleanup as part of a feature's actual completion.

## References

### Microsoft Learn

- [Feature management in .NET](https://learn.microsoft.com/dotnet/core/extensions/feature-management)

### Other

- [Martin Fowler: Feature Toggles](https://martinfowler.com/articles/feature-toggles.html)
