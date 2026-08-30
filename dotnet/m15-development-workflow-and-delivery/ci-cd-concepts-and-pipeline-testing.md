# CI/CD Concepts, and Running Tests in a Pipeline

## Definition

**Continuous Integration (CI)** means every change is automatically built and verified (tests, static analysis) as soon as it's pushed, catching integration problems immediately rather than accumulating them. **Continuous Delivery/Deployment (CD)** extends this to automatically preparing (Delivery) or actually shipping (Deployment) every verified change toward production, minimizing the manual, error-prone, and infrequent "release day" process that used to be common. Module 11 already covered running tests in CI specifically as an enforced gate; this topic covers the pipeline concept more broadly, including what happens after tests pass.

```yaml
# A typical pipeline's stages
build -> test (Module 11) -> static analysis (previous topic) -> package/containerize (previous topic)
      -> deploy to staging -> [automated or manual approval] -> deploy to production
```

## Alternatives & Trade-offs

Manual, infrequent releases (a "release day" gathering every accumulated change from weeks of work) are simpler to set up initially but concentrate risk — many changes deployed together make it hard to know which one caused a problem, and the infrequency means each release carries more accumulated, less-recently-verified change. CI/CD's frequent, small, automatically-verified releases reduce risk per release (less change, more recently tested) at the cost of the upfront investment in pipeline automation and the discipline (small PRs, per Module 15's branching topic) needed to make frequent releases actually practical.

## How It Works

### Continuous Delivery vs. Continuous Deployment — a meaningful distinction

```
Continuous Delivery:   every verified change is automatically PREPARED for release (built,
                        tested, packaged) and could be deployed at any time, but a human still
                        decides WHEN to actually trigger the deployment.
Continuous Deployment: every verified change is automatically deployed to production with NO
                        manual gate at all, as soon as it passes all automated checks.
```

Continuous Deployment requires very high confidence in the automated test suite and pipeline, since there's no human check before production — many teams land on Continuous Delivery (automated up to a manual "go" button) as the pragmatic middle ground.

### A pipeline as a sequence of gates, each narrowing what can reach the next stage

```yaml
stages:
  - build            # does it even compile?
  - test             # Module 11 — does the test suite pass?
  - static-analysis   # previous topic — does it meet the code-quality bar?
  - package           # Docker fundamentals — build the deployable artifact
  - deploy-staging     # deploy to a pre-production environment
  - smoke-test         # a quick, targeted check that staging is actually healthy post-deploy
  - deploy-production   # (with or without a manual approval gate, per Delivery vs. Deployment above)
```

Each stage is a gate — a failure at any stage stops the pipeline before the change reaches later, more consequential stages, which is exactly the enforcement mechanism Module 11's tests-in-ci.md content depends on for tests specifically.

### Pipeline speed and parallelization

```yaml
# Running independent stages in parallel, rather than strictly sequentially, when they don't
# actually depend on each other's output
jobs:
  unit-tests:      { runs-on: ubuntu-latest }
  static-analysis: { runs-on: ubuntu-latest } # can run AT THE SAME TIME as unit-tests, not after
```

A slow pipeline discourages frequent small changes (the exact discipline CI/CD is meant to enable) by making each change expensive to verify — parallelizing independent stages, and running the fastest checks first (Module 11's fast-feedback pattern), keeps the whole pipeline fast enough that frequent integration stays practical.

## Application

Design pipelines as a sequence of gates from cheapest/fastest to most expensive/slow, parallelizing independent stages where possible. Choose Continuous Delivery (automated preparation, manual deploy trigger) as a pragmatic default, moving toward full Continuous Deployment specifically once test-suite confidence genuinely supports removing the manual gate.

## Common Mistakes

- Building a pipeline so slow that developers are discouraged from the frequent, small changes CI/CD is meant to support in the first place.
- Running every pipeline stage strictly sequentially even when several are actually independent and could run in parallel.
- Adopting full Continuous Deployment before the automated test suite's coverage and reliability genuinely justify removing every manual check before production.
- Treating "we have a CI pipeline" as sufficient without actually enforcing its results as merge-blocking gates (the same enforcement point covered in Module 11).

## Common Interview Questions

### Basic
- What's the difference between Continuous Integration, Continuous Delivery, and Continuous Deployment?
- What are the typical stages in a CI/CD pipeline?

### Intermediate
- Why might a team choose Continuous Delivery over full Continuous Deployment?
- How does parallelizing independent pipeline stages improve the practicality of frequent, small releases?

### Advanced
- How would you design a pipeline's stage ordering to give the fastest possible feedback on the most common failure types?
- What level of automated test confidence would you want before recommending a team move from Continuous Delivery to full Continuous Deployment?

### Follow-up Questions
- Does Continuous Deployment require a human to ever look at a change before it reaches production?
- Can a pipeline stage failure at an early stage (e.g., build) prevent later, more expensive stages from running at all?

### Code Prediction
A pipeline runs `unit-tests` (2 minutes), `integration-tests` (8 minutes), and `static-analysis` (1 minute) strictly sequentially, totaling 11 minutes per run. If `static-analysis` and `unit-tests` don't depend on each other's output, what's the best-case total pipeline time if they're parallelized instead, assuming `integration-tests` still needs to run after both?

## Practical Tasks

- Design a CI/CD pipeline with clearly ordered stages, parallelizing at least two independent ones.
- Distinguish, for a hypothetical team's situation, whether Continuous Delivery or full Continuous Deployment is the more appropriate current choice.
- Identify a slow, strictly-sequential pipeline and redesign its stage ordering/parallelization for faster feedback.

## Readiness Criteria

Distinguish CI, Continuous Delivery, and Continuous Deployment precisely, design pipelines as fast, well-ordered, appropriately-parallelized gates, and judge when full Continuous Deployment is (or isn't yet) justified.

## References

### Microsoft Learn

- [Continuous integration and deployment concepts](https://learn.microsoft.com/devops/deliver/what-is-continuous-delivery)

### Other

- [Martin Fowler: Continuous Delivery](https://martinfowler.com/bliki/ContinuousDelivery.html)
