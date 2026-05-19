# Operating BusyNow

BusyNow is small, but it still has the same operational concerns as any public service: releases can fail, upstream dependencies can break, protections can drift, and costs can rise in the wrong places.

This document captures the main operating lessons from the project so far and the areas that still need more process and repetition.

## Operating Realities

### Small services still need release discipline

Even with a modest product surface, deploys can fail in ways that matter:

- missing GitHub variables
- missing OIDC setup
- IAM policy drift
- stale frontend assets
- runtime secret mismatches

That makes deploy and rollback clarity more important than the size of the app.

### Abuse prevention matters early

BusyNow depends on Google Places for nearby search. That creates both product value and cost exposure.

Because of that, the service needs protections around:

- edge filtering
- direct origin access
- WAF tuning
- internal header enforcement
- explicit API route allowlists

### Reliability starts with clear boundaries

Many operational problems come from ambiguity rather than raw failure:

- which route is primary for browser traffic
- which route is better for CLI verification
- which environment variable is actually used
- what should trigger a rollback
- which logs represent real business usage instead of background noise

Clean documentation reduces that ambiguity.

## What Has Helped So Far

- separate frontend and backend deployment paths
- immutable image tagging for backend releases
- CloudFront in front of both the static frontend and protected API paths
- explicit CloudFront and EKS-backed ALB routing for `/places*`, `/status*`, and `/checkin*`
- OIDC-based GitHub-to-AWS authentication
- stable anonymous request IDs for lightweight telemetry without auth
- Redis-backed cooldown coordination for repeat check-ins across EKS pods
- selective `[USAGE_EVENT]` and `[CHECKIN_EVENT]` logging in CloudWatch
- explicit rollback workflows instead of relying on ad hoc recovery
- keeping the old ECS path available only as a short-term rollback asset during the EKS soak period

These decisions do not remove operational risk, but they make the service easier to reason about and recover.

## Current Gaps

The main operating gaps today are:

- service-level review is defined but not yet routine
- environment separation is still in progress
- cost review is not yet formalized
- drills and follow-through still need more structure
- post-deploy verification of protected route behavior and Redis connectivity still needs to become routine
- operator response to burn-rate and budget trends still needs repetition
- the old ECS rollback path still needs a clear retirement decision once the EKS soak period is complete

## What Needs To Improve Next

- regular monthly SLO and error-budget review
- clearer operator habits for when degraded mode becomes too frequent
- repeatable smoke checks for `/api/places/*`, `/status*`, and `/checkin*`
- broader runbook coverage for secret rotation, cost review, and recurring failure patterns
- stronger separation between development and production environments
- clearer day-two habits around CloudWatch Insights queries for usage and check-in events

## Reliability Boundaries

The main BusyNow operating bulkheads today are:

- the expensive `/places/*` path is protected more aggressively than the static frontend path
- `/status*` and `/checkin*` share the same protected CloudFront-to-EKS-ALB flow even though their failure modes differ
- repeat check-in coordination is isolated to Redis TTL state instead of turning the persistence backend into a lock
- frontend and backend release workflows stay separate so one rollback does not automatically imply the other
- Google Places failures can stay scoped to nearby search instead of becoming a full-service outage
- the next major structural boundary is finishing the move from one live stack to clearer `dev` and `prod` responsibilities, then retiring the old ECS rollback path

## Live Stack Note

The current public app is still served from the live `dev` stack. Public backend traffic is now on EKS. The legacy ECS path is retained only as a short-term rollback asset during the current soak period.

## Related Documents

- [Current Live Architecture](architecture.md)
- [Protected API Routing](api-routing.md)
- [WAF API Behavior](waf-api-behavior.md)
- [Redis Check-In Coordination](redis-architecture.md)
- [Reliability Controls](reliability-controls.md)
- [Runbook Coverage Index](runbook-index.md)
