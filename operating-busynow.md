# Operating BusyNow

BusyNow is small on purpose, but it still behaves like a real service with real operational concerns.

## What This Project Teaches

BusyNow has been useful for practicing:

- secure GitHub-to-AWS authentication with OIDC
- separate frontend and backend deployment flows
- edge protection through CloudFront, ALB controls, and WAF
- rollback planning for containerized services
- third-party API dependency management
- budget-aware infrastructure choices

It has also been useful for practicing something that matters more at senior levels: turning a vague service into a system with clearer operating boundaries, clearer risks, and clearer upgrade paths.

## Practical Lessons So Far

### 1. Small apps still need real delivery discipline

Even when the product is simple, deploys can fail in ways that matter:

- missing GitHub variables
- missing OIDC provider setup
- IAM policy drift
- stale frontend assets
- runtime secret mismatches

BusyNow became much more valuable as a platform project once deploy and rollback paths were made explicit.

### 2. Abuse prevention matters before scale

The Google Places integration made cost exposure very real. That led to stronger design around:

- edge filtering
- direct origin protection
- WAF tuning
- internal header enforcement
- thinking about rate limits and bot traffic early

### 3. The most expensive thing is not always traffic

At low traffic, AWS request-based services often stay reasonable. Fixed costs like NAT gateways or always-on infrastructure can dominate. That changes how you think about architecture decisions when the product is still early.

### 4. Reliability starts with clarity

A lot of production pain comes from ambiguity:

- which path is primary
- which environment variable is actually used
- what triggers a rollback
- where a secret really comes from

BusyNow keeps surfacing the importance of clean documentation, not just working code.

## What Senior And Staff Growth Looks Like Here

The goal is not to make BusyNow look artificially huge. The goal is to make it show better judgment.

For this project, that means:

- documenting why one deployment model is chosen over another
- making cost tradeoffs visible instead of implicit
- defining what "healthy" means before there is major traffic
- tightening controls around the most expensive and abusable paths
- writing down failure modes and recovery steps before an incident forces them

That is the kind of work that scales beyond a single repo because it shapes how systems are operated, not just how features are shipped.

## What I Would Add To Make This Even Stronger

- a short architecture decision record history
- explicit SLO definitions
- dashboard screenshots tied to real operating signals
- sample incident writeups
- staging-to-production promotion evidence
- cost review snapshots over time

## What I’d Improve Next

- structured logging for backend requests
- CloudWatch dashboards and alarms
- staging environment promotion flow
- runbooks for common incidents
- SLOs for health and nearby search availability
- budget alarms and monthly cost review

## Why This Matters For Platform And SRE Work

BusyNow is strong portfolio material because it lets me talk about:

- system boundaries
- edge security
- delivery mechanics
- cloud IAM
- abuse resistance
- recovery paths
- production tradeoffs

That story is often more compelling than just describing app features.
