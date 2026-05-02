# BusyNow Reliability Controls

BusyNow is small, but it still has a few places where failures or abuse can get expensive quickly. The reliability model is built around two ideas:

- runbooks for the operating paths that need to be repeatable
- bulkheads that keep one class of failure from turning into a wider service problem

## Current Runbook Coverage

Release and recovery runbooks now cover the main live deployment paths:

- [Frontend Deploy](../runbooks/frontend-deploy.md)
- [Frontend Rollback](../runbooks/frontend-rollback.md)
- [Backend Deploy](../runbooks/backend-deploy.md)
- [Backend Rollback](../runbooks/backend-rollback.md)

Incident runbooks now cover the most likely operational failures in the current architecture:

- [Google Places Upstream Degradation](../runbooks/google-places-upstream-degradation.md)
- [ECS Unhealthy After Deploy](../runbooks/ecs-unhealthy-after-deploy.md)
- [Runtime Secret Or Environment Mismatch](../runbooks/runtime-secret-or-env-mismatch.md)
- [Edge Protection Drift](../runbooks/edge-protection-drift.md)

That coverage matters because the public app is still served from the live `dev` stack. The runbooks describe the system as it actually runs today instead of assuming a future-state `prod` environment that is not yet carrying traffic.

## Current Bulkheads

### Edge Bulkhead

The most expensive user path is nearby place search because it can trigger paid Google Places calls. BusyNow protects that path with layered controls:

- CloudFront in front of the public app
- WAF and rate limits around the protected API path
- internal-header enforcement between CloudFront, the ALB, and the backend

The goal is to keep valid traffic flowing without leaving the paid upstream path open to easy abuse.

### Dependency Bulkhead

BusyNow does not let every Google Places problem become a full-service outage.

- nearby search uses a per-process cache
- stale cached results can be served in degraded mode
- a call budget can fail closed with explicit cost-protection responses instead of unlimited upstream spend

That means Google degradation can stay scoped to the search path instead of turning into a broader backend incident.

### Release Bulkhead

Frontend and backend delivery are intentionally separated.

- frontend deploys move immutable build artifacts through S3 and CloudFront
- backend deploys use explicit image tags in ECS instead of relying on `latest`
- rollback paths exist for both halves of the system

This reduces the chance that a bad release requires guesswork across the whole stack at once.

### Environment Bulkhead

The current live system still runs on one stack, but the environment split is now scaffolded.

- `dev` currently backs the live public app
- `prod` now exists as a Terraform skeleton for a cleaner future separation

That is not a full bulkhead yet, but it is the next structural boundary needed to make changes safer.

## What Still Needs Work

- dashboards, alarms, and clearer service-level signals
- regular rollback drills and post-incident review habits
- broader runbook coverage for secret rotation and cost-review procedures
- finishing the move from one live stack to distinct `dev` and `prod` responsibilities

## Related Documents

- [Architecture](architecture.md)
- [Operating BusyNow](operating-busynow.md)
- [Implementation Roadmap](platform-roadmap.md)
