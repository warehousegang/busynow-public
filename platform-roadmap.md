# BusyNow Implementation Roadmap

This roadmap describes how BusyNow is planned to evolve as a product and as an operated service.

It is intentionally focused on implementation work, operating gaps, and sequencing. It is not a hiring document or a list of talking points.

## Current Baseline

BusyNow already has the core shape of the service in place:

- React frontend served through CloudFront
- static assets hosted in S3
- Express backend running on ECS Fargate behind an ALB
- Terraform-managed AWS infrastructure
- GitHub Actions deployment workflows
- WAF and origin protection around the API path
- Google Places integration for nearby search
- crowd check-ins and derived place status

That baseline is enough to run the product publicly. The roadmap below is about making it more reliable, easier to operate, and clearer to evolve.

## Next Planned Delivery Slice

The next visible BusyNow iteration is expected to combine one product-facing improvement with one runtime-hardening improvement:

- a fuller maps view for exploring nearby places spatially
- a hardening pass on the places API path so upstream failures and traffic spikes degrade more safely

This work is planned soon, but it is not documented as already implemented.

### Upcoming Product Work: Maps View

Goal:
Add a clearer map-based way to browse nearby places without making the product feel heavy or overbuilt.

Planned work:

- improve the public maps experience so nearby results can be explored spatially, not only as a list
- make transitions between list and map exploration feel intentional
- keep place status visibility clear inside the map experience
- reduce empty or confusing states when map data is partial or still loading

Done when:

- the maps view feels like a first-class part of discovery
- list and map browsing support each other instead of competing
- users can understand nearby context faster

### Upcoming Platform Work: Places Path Hardening

Goal:
Make nearby search more resilient, more observable, and safer to operate under load or upstream failure.

Planned work:

- add a shared Redis cache compatible with ElastiCache for nearby places responses
- use rounded-coordinate cache keys with TTL support and stale-response fallback support
- replace any process-local caching on the places path
- move rate limiting earlier in the flow so the request path becomes `API -> Rate Limit -> Cache -> Google`
- enforce per-IP or per-user limits and prefer stale cached responses when limits are hit
- return a degraded empty response when Google is unavailable and no cached result exists
- add concurrency protection around outbound Google requests with queueing or rejection behavior
- fall back to stale cache when concurrency limits are exhausted
- emit structured JSON logs that are CloudWatch-ready for cache hits, Google calls, failures, rate limits, and latency
- require the internal `x-internal-key` header on API requests and keep that aligned with ALB listener protections

Done when:

- Google API failures do not fully break nearby search when stale data exists
- load spikes are less likely to trigger runaway third-party cost
- degraded behavior is intentional and externally visible
- logs make request-path behavior easier to understand during debugging

## Roadmap Principles

- Keep the user-facing product simple while improving operating quality
- Prefer small, reversible changes over large rewrites
- Add complexity only when it clearly improves safety, clarity, or reliability
- Treat cost and abuse prevention as part of the implementation work
- Document each stage well enough that another engineer could operate it

## Phase 1: Stabilize The Current Public Service

Goal:
Make the current single-environment service more predictable to run day to day.

Planned work:

- clean up public and internal documentation so it matches the current implementation
- tighten deployment steps for frontend and backend releases
- verify rollback paths for both frontend assets and backend task definitions
- document required runtime configuration and external dependencies
- reduce rough edges in the public product experience on desktop and mobile

Done when:

- release steps are explicit and repeatable
- rollback is documented and tested
- public docs describe the system as it actually exists today

## Phase 2: Improve Product Data Quality

Goal:
Make nearby results and crowd signals more useful without making the app heavy.

Planned work:

- improve how nearby places are selected and filtered
- refine how place details are stored and updated after search or check-in
- improve crowd-signal quality when a place has little or no recent data
- make status freshness easier to understand in the UI
- reduce confusing empty states and unclear result states

Done when:

- users can more consistently find relevant places
- status results are easier to interpret
- low-data scenarios feel intentional instead of unfinished

## Phase 3: Add Better Visibility Into Runtime Health

Goal:
Make it easier to understand whether the service is healthy and what changed when it is not.

Planned work:

- structured backend request logging
- dashboards for traffic, latency, errors, and deployment status
- basic alarms for unhealthy targets and elevated failure rates
- uptime checks for the homepage and health endpoint
- clearer separation between application failures and upstream dependency failures

Done when:

- routine health checks do not depend on manual guesswork
- regressions can be detected quickly after deploys
- debugging starts from signals instead of intuition

## Phase 4: Separate Environments Cleanly

Goal:
Move from a single live stack toward clearer environment boundaries.

Planned work:

- finish the split between `dev` and `prod` infrastructure
- define environment-specific configuration and secrets cleanly
- promote immutable frontend and backend artifacts forward instead of rebuilding per environment
- add environment protection rules around deploy workflows
- make it clearer which changes are safe to test only in development first

Done when:

- development and production have distinct responsibilities
- releases move forward intentionally between environments
- environment drift is easier to spot and correct

## Phase 5: Strengthen Security And Abuse Controls

Goal:
Protect the most expensive and most exposed paths without overcomplicating the runtime.

Planned work:

- review direct-origin protections and internal-header enforcement
- keep WAF rules aligned with observed traffic patterns
- improve rate-limiting strategy for expensive endpoints
- document secret rotation expectations and ownership
- review IAM and deployment permissions for least privilege

Done when:

- the riskiest paths are intentionally protected
- abuse controls are documented and measurable
- security-sensitive configuration is easier to reason about

## Phase 6: Make Operations More Repeatable

Goal:
Turn operational knowledge into documented procedures.

Planned work:

- write runbooks for deploy, rollback, and common failure cases
- document expected responses to upstream outages and bad releases
- add lightweight release verification and smoke checks
- create a simple incident review template
- record architecture and operations decisions that are easy to forget later

Done when:

- common incidents have a written response path
- operating the service does not depend on memory alone
- future changes have a clearer paper trail

## Phase 7: Improve Cost Awareness And Sustainability

Goal:
Keep the service practical to run as usage and infrastructure maturity increase.

Planned work:

- review fixed AWS costs and high-risk variable costs
- add budget alerts where they provide useful signal
- track which protections reduce unnecessary paid API traffic
- review whether current infrastructure choices still fit the product stage
- capture cost tradeoffs in architecture decisions

Done when:

- cost surprises are less likely
- infrastructure changes are easier to justify
- reliability work and budget work support each other instead of competing

## Near-Term Priorities

The next practical milestones are:

1. keep the public docs aligned with the current implementation and near-term plan
2. deliver the next maps view experience
3. harden the places request path with shared cache, graceful degradation, and earlier protection controls
4. add basic runtime visibility through logs, dashboards, and alarms
5. complete the environment split so `dev` and `prod` are no longer blurred
6. write the first runbooks for deploy, rollback, and upstream failure handling

## What This Roadmap Is Not

This roadmap is not a promise to add every possible platform feature.

BusyNow does not need complexity for its own sake. The intention is to keep the service understandable, improve it in public over time, and add operating depth only where it produces real value.
