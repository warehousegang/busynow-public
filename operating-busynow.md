# Operating BusyNow

BusyNow is small, but it still has the same operational concerns as any public service: releases can fail, upstream dependencies can break, protections can drift, and costs can rise in the wrong places.

This document captures the main operating lessons from the project so far and the areas that still need more process and automation.

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
- rate limits on expensive paths

### Fixed infrastructure costs matter

At low traffic, fixed AWS costs can matter more than request volume. Components like load balancers, NAT, or always-on runtime services can shape architecture decisions before scale does.

### Reliability starts with clear boundaries

Many operational problems come from ambiguity rather than raw failure:

- which route is primary
- which environment variable is actually used
- what triggers a rollback
- where a secret is sourced from
- which failure should surface to the user versus stay internal

Clean documentation reduces that ambiguity.

## What Has Helped So Far

- separate frontend and backend deployment paths
- immutable image tagging for backend releases
- CloudFront in front of both the static frontend and protected API paths
- OIDC-based GitHub-to-AWS authentication
- explicit rollback workflows instead of relying on ad hoc recovery

These decisions do not remove operational risk, but they make the service easier to reason about and recover.

## Current Gaps

The main operating gaps today are:

- observability is still thinner than it should be
- incident response steps are not fully documented
- environment separation is still in progress
- cost review is not yet formalized
- degraded-mode behavior for upstream issues can be clearer

## What Needs To Improve Next

- structured backend logging
- dashboards and alarms for core service health
- clearer runbooks for deploy, rollback, and upstream dependency failure
- stronger separation between development and production environments
- lightweight release verification after deploys
- regular cost and protection reviews for the most exposed paths

## Operating Direction

The direction for BusyNow is straightforward:

1. keep the runtime simple
2. make deployments safer and easier to verify
3. improve visibility into failures and regressions
4. document common operating paths
5. add more process only where it reduces real risk

## Related Documents

- [Architecture](architecture.md)
- [Implementation Roadmap](platform-roadmap.md)
- [Engineering Principles And Tradeoffs](engineering-principles.md)
