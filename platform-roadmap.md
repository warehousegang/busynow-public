# BusyNow Implementation Roadmap

This roadmap describes how BusyNow is planned to evolve as a product and as an operated service.

It is intentionally focused on implementation work, operating gaps, and sequencing. It is not a hiring document or a list of talking points.

## Current Baseline

BusyNow already has the current live service shape in place:

- React frontend served through CloudFront
- static assets hosted in S3
- Express backend running on EKS behind an ALB ingress
- explicit CloudFront routing for `/places*`, `/api/places*`, `/status*`, and `/checkin*`
- frontend WAF Bot Control with stricter protection on `/places/*`
- production delivery through Helm, Argo CD, immutable ECR images, and GitHub Actions
- HPA pod scaling with Karpenter-managed elastic node capacity
- stable anonymous browser IDs and selective structured usage telemetry
- Redis-backed distributed check-in dedupe with graceful degradation
- optional persistence backends depending on environment and runtime configuration
- Google Places integration for nearby search
- deploy, rollback, and first-line incident runbooks for the live system
- a production-owned public service with ephemeral development infrastructure

That baseline is enough to run the product publicly. The roadmap below is about making it easier to operate, verify, and evolve.

## Roadmap Principles

- Keep the user-facing product simple while improving operating quality
- Prefer small, reversible changes over large rewrites
- Add complexity only when it clearly improves safety, clarity, or reliability
- Treat cost and abuse prevention as part of the implementation work
- Document each stage well enough that another engineer could operate it

## Phase 1: Stabilize The Current Public Service

Goal:
Make the production service more predictable to operate day to day.

Planned work:

- keep public and internal documentation aligned with the current live implementation
- tighten release verification steps for frontend and backend changes
- verify rollback paths for both frontend assets and backend deployment revisions
- document required runtime configuration and external dependencies
- make route-level smoke checks more routine after deploys
- exercise GitOps drift, rollback, pod-failure, and capacity-scaling procedures regularly

## Phase 2: Improve Product Data Quality

Goal:
Make nearby results and crowd signals more useful without making the app heavy.

Planned work:

- improve how nearby places are selected and filtered
- refine how place details are stored and updated after search or check-in
- improve crowd-signal quality when a place has little or no recent data
- make status freshness easier to understand in the UI
- reduce confusing empty states and unclear result states

## Phase 3: Deepen Runtime Visibility

Goal:
Make it easier to understand whether the service is healthy and what changed when it is not.

Planned work:

- expand current selective structured logging into more routine operator queries
- add dashboards for traffic, latency, errors, and deployment status
- add basic alarms for unhealthy targets and elevated failure rates
- improve saved CloudWatch Insights queries for usage and check-in events
- connect alert signals to documented first-response procedures
- make post-deploy verification less dependent on memory

## Phase 4: Mature Environment Promotion

Goal:
Keep production stable while making temporary development environments easier to create, verify, and remove.

Planned work:

- keep `prod` as the sole owner of the public service
- preserve `dev` as an ephemeral, non-production sandbox
- codify repeatable development environment create and teardown workflows
- promote immutable frontend and backend artifacts without rebuilding them per environment
- add environment protection rules around deploy workflows
- make configuration and secret boundaries explicit

## Phase 5: Strengthen Security And Abuse Controls

Goal:
Protect the most expensive and most exposed paths without overcomplicating the runtime.

Planned work:

- keep WAF rules aligned with observed traffic patterns
- review whether `/places/*` Bot Control policy should stay strict or become more selective
- document trusted verification traffic options more clearly
- review direct-origin protections and internal-header enforcement
- review IAM, ingress, and deployment permissions for least privilege

## Phase 6: Make Operations More Repeatable

Goal:
Turn operational knowledge into documented procedures.

Planned work:

- extend the current runbook set with more recurring failure, secret, and cost procedures
- add lightweight release verification and smoke checks
- create a simple incident review template
- record architecture and operations decisions that are easy to forget later
- make Redis, routing, WAF, and EKS verification part of normal deploy hygiene

## Phase 7: Improve Cost Awareness And Sustainability

Goal:
Keep the service practical to run as usage and infrastructure maturity increase.

Planned work:

- review fixed AWS costs and high-risk variable costs
- add budget alerts where they provide useful signal
- track which protections reduce unnecessary paid API traffic
- review whether the current infrastructure choices still fit the product stage
- capture cost tradeoffs in architecture decisions

## Near-Term Priorities

The next practical milestones are:

1. keep the public docs aligned with the current implementation
2. finish stabilizing frontend and backend deployment workflows
3. add more runtime visibility through logs, dashboards, and alarms
4. make ephemeral development creation and teardown more repeatable
5. broaden runbook coverage and establish smoke-check, rollback, and incident-review cadence

## What This Roadmap Is Not

This roadmap is not a promise to add every possible platform feature.

BusyNow does not need complexity for its own sake. The intention is to keep the service understandable, improve it in public over time, and add operating depth only where it produces real value.
