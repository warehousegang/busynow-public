# BusyNow Engineering Principles And Tradeoffs

This document describes the main engineering choices behind BusyNow and the tradeoffs that shaped them.

## Core Principles

### 1. Reversibility Over Heroics

Deployments should be easy to understand, easy to verify, and easy to undo.

In practice, that means:

- explicit image tags instead of relying on `latest`
- separate frontend and backend deploy paths
- rollback workflows treated as a normal part of delivery

### 2. Protect Expensive Dependencies With Bulkheads

Google Places improves the product, but it also creates cost and abuse risk. That makes edge protection part of the application design, not a separate cleanup task.

In practice, that means:

- CloudFront in front of the application
- explicit backend origin routing instead of relying on the SPA fallback path
- WAF and Bot Control around the browser-facing search route
- cached or degraded behavior before uncontrolled upstream spend
- attention to traffic patterns that can trigger unnecessary paid API usage

### 3. Favor Simple Runtime Paths

The runtime should stay simple enough that failure modes are understandable. Additional complexity is only useful when it makes the system safer or easier to operate.

In practice, that means:

- static frontend hosting
- a single ECS service instead of heavier orchestration
- Redis only for lightweight TTL-backed coordination, not as a general cache or auth layer

### 4. Match Controls To Real Risks

BusyNow does not need every possible control at once, but it does need the controls that address the risks it actually has.

Current risks include:

- direct origin abuse
- bot traffic causing paid upstream API calls
- leaked or misused credentials
- accidental release breakage
- noisy logs that hide the business-relevant traffic patterns

### 5. Use Lightweight Identity When Auth Is Not The Goal

BusyNow needs enough request identity to reason about user behavior and duplicate check-ins, but it does not need an account system for the current product flow.

In practice, that means:

- the browser persists an anonymous UUID in `bn_user_id`
- every browser request includes `x-bn-user-id`
- the ID is used for telemetry and check-in dedupe only

### 6. Treat Cost As An Engineering Constraint

For a small service, cost problems can be as disruptive as availability problems. Budget awareness is part of operating the system responsibly.

In practice, that means:

- reviewing fixed-cost AWS components
- being intentional about WAF and bot-control settings
- understanding where architecture, not traffic, is driving spend

## Key Tradeoffs

### Edge Security vs Convenience

Current preference:

- stricter protection on `/places/*`
- CLI and scripted verification traffic on `/api/places/*`

Why:

- protects the browser-facing paid search path
- preserves an operational path for testing and verification
- keeps the difference intentional instead of accidental

Tradeoff:

- some curl and HTTP-library requests to `/places/*` can look like routing failures even when routing is correct
- the public docs must explain that behavior clearly so it is not mistaken for a CloudFront defect

### Simplicity vs Safer Rollouts

Current preference:

- ECS rolling deployments

Why:

- simpler to operate
- fewer moving parts
- easier to reason about at the current size of the service

### Lightweight Coordination vs Broader State Layers

Current preference:

- Redis TTL keys only for repeat check-in coordination

Why:

- ECS tasks do not share process memory
- duplicate suppression needs a shared ephemeral state layer
- the problem does not justify a heavier session or locking system

Tradeoff:

- Redis adds one more dependency to operate
- the app still needs to degrade safely when Redis is unavailable

## Decision Style

BusyNow is intended to evolve through small, reviewable changes.

That means:

- choosing clear systems over impressive ones
- documenting why a control or workflow exists
- delaying complexity until there is a concrete operational reason for it
- preferring tools and patterns that are easy to explain and maintain

## Related Documents

- [Current Live Architecture](architecture.md)
- [Protected API Routing](api-routing.md)
- [WAF API Behavior](waf-api-behavior.md)
- [Redis Check-In Coordination](redis-architecture.md)
- [Reliability Controls](reliability-controls.md)
- [Operating BusyNow](operating-busynow.md)
