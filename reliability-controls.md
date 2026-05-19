# BusyNow Reliability Controls

BusyNow is small, but it still has a few places where failures or abuse can get expensive quickly. The reliability model is built around two ideas:

- bulkheads that keep one class of failure from turning into a wider service problem
- repeatable operating notes and runbooks for the paths that matter most

## Current Coverage

The internal runbook set now covers:

- frontend deploy and rollback
- EKS backend deploy and validation
- legacy ECS rollback path during soak
- Google Places upstream degradation
- runtime secret or environment mismatch
- edge protection drift
- persistence backend unavailable

That coverage matters because the public app is still served from the live `dev` stack, but the backend path now runs on EKS instead of ECS. The operating notes describe the system as it actually runs today instead of assuming a future-state `prod` environment that is not yet carrying traffic.

## Current Bulkheads

### Edge Bulkhead

The most expensive user path is nearby place search because it can trigger paid Google Places calls. BusyNow protects that path with layered controls:

- CloudFront in front of the public app
- WAF and Bot Control around the browser-facing `/places/*` path
- internal-header enforcement between CloudFront, the ALB, and the backend
- explicit CloudFront and ALB allowlists for `/places*`, `/status*`, and `/checkin*`

The goal is to keep valid traffic flowing without leaving the paid upstream path open to easy abuse.

### Dependency Bulkhead

BusyNow does not let every Google Places problem become a full-service outage.

- nearby search is the only path that directly triggers Google Places
- degraded or cost-protected behavior can stay scoped to the search path
- `/status*` and `/checkin*` remain separate from the paid upstream lookup path

### Coordination Bulkhead

BusyNow uses Redis as a very small shared coordination surface for check-ins.

- Redis stores only ephemeral `checkin:{place_id}:{user_id}` cooldown keys
- repeated check-ins refresh a 30 minute TTL instead of creating unbounded duplicate writes
- the backend degrades safely when Redis is unavailable, so Redis failure does not become a full API outage

This keeps horizontally scaled EKS pods consistent without turning Redis into a general-purpose cache or source of truth.

### Observability Bulkhead

- usage telemetry is filtered to business-relevant `/places*` and `/checkin*` routes
- structured single-line `[USAGE_EVENT]` and `[CHECKIN_EVENT]` logs are queryable in CloudWatch Insights
- noisy paths like `/health` are intentionally excluded so operational queries stay focused

### Release Bulkhead

Frontend and backend delivery are intentionally separated.

- frontend deploys move immutable build artifacts through S3 and CloudFront
- backend deploys use explicit image tags on the EKS `Deployment` instead of relying on `latest`
- rollback paths exist for both halves of the system

### Environment Bulkhead

The current live system still runs on one stack, but the environment split is now scaffolded.

- `dev` currently backs the live public app
- `prod` exists as a Terraform skeleton for a cleaner future separation

That is not a full bulkhead yet, but it is the next structural boundary needed to make changes safer.

## What Still Needs Work

- regular SLO review habits and monthly budget retrospectives
- regular rollback drills and post-incident review habits
- broader runbook coverage for secret rotation and cost-review procedures
- repeatable post-deploy checks for protected API path reachability and Redis connectivity
- finishing the move from one live stack to distinct `dev` and `prod` responsibilities
- retiring the old ECS rollback path once the EKS soak period is complete

## Related Documents

- [Current Live Architecture](architecture.md)
- [Protected API Routing](api-routing.md)
- [WAF API Behavior](waf-api-behavior.md)
- [Redis Check-In Coordination](redis-architecture.md)
- [Operating BusyNow](operating-busynow.md)
- [Runbook Coverage Index](runbook-index.md)
