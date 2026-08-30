# Basic Release and Rollback Thinking

## Definition

This closing topic ties together the whole module's delivery content into the practical question every release ultimately asks: if this goes wrong, how would we know, and how would we recover? A **release strategy** is the shape a deployment takes (all-at-once, gradual, canary); **rollback thinking** is having a concrete, pre-planned answer for "undo this" before it's actually needed, not improvised during an active incident.

```
Deployment shapes, roughly in order of increasing safety and complexity:
  All-at-once:       every instance updated simultaneously — simplest, riskiest
  Rolling:            instances updated gradually, old and new versions briefly coexist
  Blue-green:         a full second environment stood up, traffic switched once verified
  Canary:              a small fraction of traffic routed to the new version first, expanded
                       gradually while watching metrics (Module 13) before a full rollout
```

## Alternatives & Trade-offs

An all-at-once deployment is simplest to reason about and fastest to execute, but means a bug affects 100% of traffic immediately with no gradual warning, and requires reverting/redeploying to recover. Canary and blue-green deployments cost more infrastructure and pipeline complexity, but let a problem be caught while affecting only a small fraction of traffic (canary) or let a full rollback happen by simply switching traffic back (blue-green) — trading complexity for a much smaller worst-case blast radius.

## How It Works

### Canary deployment — catching a problem before it's everyone's problem

```
1. Deploy the new version alongside the old, routing only 5% of traffic to it
2. Watch error rates, latency (Module 13's metrics) for the canary specifically, compared to
   the still-running old version handling the other 95%
3. If healthy, gradually increase the canary's traffic share; if not, route back to 0% instantly
```

This is the deployment-level version of the feature flag's percentage-rollout idea — same principle (limit exposure, watch, expand or retreat), applied at the infrastructure/routing level instead of an application-level conditional.

### Blue-green deployment — instant rollback via traffic switching

```
Blue:  the currently-live environment, serving 100% of traffic
Green: a full second environment running the NEW version, verified healthy but not yet
       receiving real traffic
Cutover: switch the load balancer/router to send traffic to Green instead of Blue
Rollback: if Green has a problem, switch traffic back to Blue INSTANTLY — Blue never stopped
          running, so this is a traffic-routing change, not a redeploy
```

The rollback here is about as fast as a rollback can be, precisely because the previous version was never actually torn down — it just stopped receiving traffic, and can start receiving it again immediately.

### Having an actual rollback plan before deploying, not improvising one during an incident

```
Before deploying: what EXACTLY would "roll back" mean for this specific change?
  - Just redeploy the previous code version? (usually straightforward, per the migrations topic)
  - Does this deployment include a database migration that complicates or prevents a clean
    code-only rollback? (the migrations topic's core concern)
  - Is there a feature flag that could instantly disable the risky part without any redeploy
    at all? (the feature-flags topic)
```

Answering this *before* deploying, as part of planning the release, means an actual incident doesn't require inventing a rollback plan under pressure — it means executing one already decided on calmly beforehand.

### Monitoring as the detection half of "if this goes wrong, how would we know"

```
A release without corresponding monitoring/alerting (Module 13) has no fast detection
mechanism at all — a problem might only surface once users start complaining, far later than
a metric-based alert would have caught it.
```

## Application

Choose a deployment shape (all-at-once, rolling, canary, blue-green) proportional to the change's actual risk — not every release needs canary/blue-green's full complexity, but a genuinely risky change benefits from it. Before every deployment, have an explicit, concrete answer for what rollback actually means for that specific change, accounting for any accompanying migration. Pair every release with the monitoring needed to actually detect a problem quickly.

## Common Mistakes

- Deploying without having a concrete rollback plan decided in advance, leaving the team to improvise one under pressure during an actual incident.
- Using an all-at-once deployment for a genuinely risky change where a canary or gradual rollout would have caught a problem while affecting far fewer users.
- Forgetting that a deployment including a data-transforming migration may not support a clean code-only rollback, discovering this only once a rollback is actually needed.
- Releasing without corresponding monitoring/alerting in place, relying on user complaints as the only detection mechanism for a problem.

## Common Interview Questions

### Basic
- What's the difference between a rolling deployment, a canary deployment, and a blue-green deployment?
- Why should a rollback plan be decided before a deployment, not during an incident?

### Intermediate
- How does a canary deployment limit the blast radius of a bad release compared to an all-at-once deployment?
- Why does blue-green deployment enable an especially fast rollback?

### Advanced
- How would you decide the appropriate deployment shape for a given change, based on its actual risk?
- How does a database migration accompanying a deployment complicate the rollback plan, and how would you design around that complication?

### Follow-up Questions
- Is canary deployment always worth its added infrastructure complexity?
- Does having a rollback plan mean a deployment is risk-free?

### Code Prediction
A team deploys a risky change all-at-once to 100% of instances at once, with a migration that renames a column the old code still reads. A production bug is discovered ten minutes later. What are this team's actual rollback options, given the migration's incompatibility with the old code, and how would a canary deployment combined with the expand/contract migration pattern have changed this situation?

## Practical Tasks

- Design a canary rollout plan for a risky change, including the specific metrics that would gate expanding traffic to it further.
- Write an explicit rollback plan for a hypothetical deployment before it happens, including how any accompanying migration affects that plan.
- Compare the rollback speed and mechanism for an all-at-once deployment versus a blue-green deployment for the same hypothetical change.

## Readiness Criteria

Choose deployment shapes proportional to actual risk, have a concrete rollback plan decided before every deployment rather than improvised during an incident, and account for how migrations affect rollback options.

## References

### Microsoft Learn

- [Deployment strategies overview](https://learn.microsoft.com/devops/deliver/what-is-continuous-delivery)

### Other

- [Martin Fowler: BlueGreenDeployment](https://martinfowler.com/bliki/BlueGreenDeployment.html)
- [Martin Fowler: CanaryRelease](https://martinfowler.com/bliki/CanaryRelease.html)
